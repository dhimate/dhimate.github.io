---
title: "Using Web Services Client adapter in Apama"
date: 2016-08-24
tags:
 - Apama
categories:
 - Programming 
toc: true
---
Software AG Apama provides a SOAP-based Web Services Client adapter to invoke web services from your Apama application. In this post, we will see how to use this adapter in your application.

We will use a free web service available at  CDYNE (link) for our application. This service has multiple operations, out of which we will use GetCityWeatherByZIP operation.

Below are the sample request and response messages for this operation, which we will use to describe our event definition in Apama.

Request XML
{{< codecaption lang="xml" title="Request XML" >}}
    <soap:envelope 
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" 
    xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <soap:body>
        <GetCityWeatherByZip xmlns="http://ws.cdyne.com/WeatherWS/">
        <Zip>43209</Zip>
        </GetCityWeatherByZip>
    </soap:body>
    </soap:envelope>
{{< /codecaption >}}

Response XML
{{< codecaption lang="xml" title="Response XML" >}}    <soap:Envelope 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <GetCityWeatherByZIPResponse 
        xmlns="http://ws.cdyne.com/WeatherWS/">
        <GetCityWeatherByZIPResult>
            <Success>true</Success>
            <ResponseText>City Found</ResponseText>
            <State>OH</State>
            <City>Columbus</City>
            <WeatherStationCity>Columbus</WeatherStationCity>
            <WeatherID>14</WeatherID>
            <Description>Cloudy</Description>
            <Temperature>63</Temperature>
            <RelativeHumidity>83</RelativeHumidity>
            <Wind>CALM</Wind>
            <Pressure>29.93R</Pressure>
            <Visibility/>
            <WindChill/>
            <Remarks/>
        </GetCityWeatherByZIPResult>
        </GetCityWeatherByZIPResponse>
    </soap:Body>
    </soap:Envelope>

{{< /codecaption >}}

Let's begin by creating a new Apama project. We will name it WeatherWebServiceDemo.

While creating the project using 'New Apama Project Wizard', select 'Web Services Client Adapter' bundle

![img](/img/2016/ApamaWS_1.PNG)



Now let's describe the event definitions. We will define events such that their structure corresponds directly to the XML structure of the web service message shown above. Such type of mapping is called as 'Convention based' mapping where Apama automatically converts the event instance to the request XML structure of the web service, and also converts the web service response XML to an event instance.

The event definition will look like this:

{{< codecaption lang="java" title="" >}}
    event GetCityWeatherByZIPType {
        string ZIP;
    }

    event GetCityWeatherByZIPResultType {
        string Success;
        string ResponseText;
        string State;
        string City;
        string WeatherStationCity;
        string WeatherID;
        string Description;
        string Temperature;
        string RelativeHumidity;
        string Wind;
        string Pressure;
        string Visibility;
        string WindChill;
        string Remarks;
    }

    event GetCityWeatherByZIPResponseType {
        GetCityWeatherByZIPResultType GetCityWeatherByZIPResult;
    }

    event MyRequest {
        GetCityWeatherByZIPType GetCityWeatherByZIP;
    }

    event MyResponse {
        GetCityWeatherByZIPResponseType GetCityWeatherByZIPResponse;
    }

    event MyZIP {
        string ZIP;
    }

{{< /codecaption >}}

Out of these the MyRequest and MyResponse event definitions are wrapper types required for Convention based mapping, whereas MyZIP is our custom event definition which we will use to request weather information for a zip code.

After describing the event definitions, we can import the WSDL and map the event definitions to web service input and output. To do that, open the adapter instance file located in Adapters->WebServices Client Adapter -> instance1 in your project and click on + (plus) to add the operation name, it will bring up the following screen to configure your web service.   

![img](/img/2016/ApamaWS_1_Import.PNG)


In this screen click on 'Add' operation, and then click on 'Create New' to bring the following screen to import the WSDL and select the operation that you want to use. 

![img](/img/2016/ApamaWS_2_Import.PNG)

We will only import GetCityWeatherByZIP operation for our application as shown below. 

![img](/img/2016/ApamaWS_3_Import.PNG)


After importing the operation select the input and output event types corresponding to the input and output of the operation

![img](/img/2016/ApamaWS_4_Import.PNG)

Once input and output event for the web service is selected, you can perform the mapping of fields to web service parameters

In the input mapping it's important to select the 'Convert to XML' transformation to convert the event structure to XML. You can select this transformation by using 'Add computed node' option in your input mapping. 

![img](/img/2016/ApamaWS_input_mapping.PNG)

The output mapping is straightforward as shown below.

![img](/img/2016/ApamaWS_output_mapping.PNG)

Once the mapping is completed you can write your monitor script (EPL) program to call the weather service and then identify the pattern in the data.

Following simple monitor script program will just print the weather information received from web service.

In this script, whenever the correlator engine detects MyZIP event in the queue, it will route the MyRequest event. MyRequest event will in turn call the web service to get the weather information using web service client adapter and receive the response in the form of MyResponse event. Whenever MyResponse event is detected, our script prints the weather information in the log.

{{< codecaption lang="java" title="" >}}
    monitor WeatherServiceMonitor {
        MyRequest myRequest;
        MyResponse myResponse;
        MyZIP myZIP;

        action onload() {

            on all MyZIP(): myZIP {
                print "Received request for ZIP code : " + myZIP.ZIP;
                print "Preparing to call the service " ;
                myRequest.GetCityWeatherByZIP.ZIP := myZIP.ZIP;
                route myRequest;            
            }

            on all MyResponse(): myResponse {
                print 
                    myResponse.GetCityWeatherByZIPResponse.
                    GetCityWeatherByZIPResult.ResponseText 
                + " " 
                +     myResponse.GetCityWeatherByZIPResponse.
                    GetCityWeatherByZIPResult.City
                + " " 
                +     myResponse.GetCityWeatherByZIPResponse.
                    GetCityWeatherByZIPResult.Temperature
                + " " 
                +     myResponse.GetCityWeatherByZIPResponse.
                    GetCityWeatherByZIPResult.Description
                + " " 
                +     myResponse.GetCityWeatherByZIPResponse.
                    GetCityWeatherByZIPResult.Wind; 
            }
        }
    }

{{< /codecaption >}}

 Let's fetch the weather for zip code 43209
{{< codecaption lang="shell" title="" >}}
    C:\SoftwareAG\Apama\bin>engine_send
    MyZIP("43209");
{{< /codecaption >}}

![img](/img/2016/ApamaWS_result.PNG)


We can invoke the web service periodically to fetch the weather information using 'on all wait() ' as shown below. This code snippet will route the MyZIP event every minute (60 seconds) to fetch the weather information.

{{< codecaption lang="java" title="" >}}
    on all wait (60.0) {
      MyZIP("43209"); 
    }
{{< /codecaption >}}

Instead of logging the information to a file we can actually use Apama Dashboard to visually monitor the weather fluctuation and any patterns in the data. I will explain how to do that in another post.
