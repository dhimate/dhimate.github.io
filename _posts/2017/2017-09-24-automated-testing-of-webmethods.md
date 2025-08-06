---
title: "Automated Testing of Webmethods Services with REST Assured"
date: 2017-09-24
layout: post
tags:
 - webMethods
categories:
 - Programming
toc: true
---
While testing REST APIs for one of my projects, I found REST Assured. It was perfect, as it took care of low level HTTP calls under the hood, and provided a high-level, easy to use framework to write tests. Not only REST Assured works really well to test webMethods flow services, you can also run these tests as part of Continuous Integration process through Gradle.

You will need following things on your machine. Make sure these are working without any issues.

- Java JDK 1.8 ([link](http://www.oracle.com/technetwork/java/javase/downloads/index.html)).  Set JAVA_HOME. And add JAVA_HOME to your PATH
- IntelliJ IDEA ([link](http://www.jetbrains.com/idea/download/))

We will write automated tests for a simple flow service that adds two numbers.

  ![Flow_Service](/img/2017/Flow_Service.gif)

Follow following steps to create automated tests.

- Open IntelliJ IDEA and create a new project

  ![Create_Project](/img/2017/Create_Project.gif)

- Add following lines in your build.gradle file

{{< codecaption lang="java" title="" >}}
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    testCompile group: 'org.hamcrest', name: 'hamcrest-library', version: '1.3'
    testCompile group: 'org.hamcrest', name: 'hamcrest-core', version: '1.3'
    testCompile 'io.rest-assured:rest-assured:3.0.3'
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.0' //version required
} 
{{< /codecaption >}}

  ![Update_BuildGradle](/img/2017/Update_BuildGradle.gif)

- Create new package for our test cases

  ![Create_Package](/img/2017/Create_Package.gif)

- Create new Java class for our tests. 

{{< codecaption lang="java" title="" >}}
package com.dhimate.wm;

import org.junit.Before;
import org.junit.Test;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import java.util.HashMap;
import static io.restassured.RestAssured.given;
import static io.restassured.RestAssured.basic;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.equalTo;

public class AddNumbersTest {

	@Before
	public void setUp() throws Exception {
		RestAssured.baseURI = "http://localhost:5555/invoke";
		RestAssured.authentication = basic("Administrator", "manage");
	}

	@Test
	public void AddNumbersTestSuccess() {
		String requestBody = "{" +
                    "\"num1\":\"10\"," +
                    "\"num2\":\"20\"" +
                    "}";

		Response response = given().
                	header("Content-Type","application/json").
	                body(requestBody).
       			put("Yogesh.flow/addNumbers");

        		response.getBody().prettyPrint();

        		HashMap responseBody = response.jsonPath().get("");

        		assertThat("status code", response.getStatusCode(), equalTo(200));
        		assertThat("sum", responseBody.get("sum"), equalTo("30"));
    	}
}
{{< /codecaption >}}

Here we are just taking advantage of Integration Server's built in content handler for Content Type - 'application/json'. For the flow service available on Integration Server, we can simply pass JSON request using REST Assured framework. Integration Server will take care of converting this to IDoc and subsequently process it to return a JSON response back to your test case.

- Run the tests using Gradle wrapper

Gradle wrapper provides a convenient and easy to use CLI to run your tests. Using Gradle wrapper you can quickly associate your REST Assured tests with your Continuous Integration process.

  ![Gradle_Wrapper](/img/2017/Gradle_Wrapper.gif)

Good thing about using REST Assured and Hamcrest to test your webMethods flow services is, it is completely FREE.  You don't need to buy an expensive automated testing solution to test your flow services.

Further resources to read

  1. REST Assured has comprehensive [documentation](https://github.com/rest-assured/rest-assured/wiki/GettingStarted) available to get you started.
  2. Excellent tutorial on [Gradle](http://www.vogella.com/tutorials/Gradle/article.html) and [Hamcrest](http://www.vogella.com/tutorials/Hamcrest/article.html) by Vogella.

