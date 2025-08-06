---
title: "Externalizing Queries in Mule Application"
date: 2021-01-09
layout: post
tags:
 - MuleSoft
categories:
 - Programming
draft: false
description: "Externalize SQL, SOQL and other queries in MuleSoft"
---
I like to keep my code clean, understandable, and readable. It helps the next person who will take over my work. I try my best to avoid any trouble reading, maintaining, and updating the code.
There are recommended and well known best practices about organizing Mule code. It involves better structuring of the project, maintaining configuration files, hiding or securing sensitive information like passwords and API keys, transaction management, or reconnection and caching strategies.

The lesser-known trick is about externalizing SOQL and SQL queries. Before I knew this trick I organized my SQL queries in properties files, and used spring placeholders like `${account.query}`  or dataweave `Mule::p` function to inject or read the queries in the code. It is not an elegant approach. It made the properties file difficult to manage and read. Then I discovered `${file::}` notation to inject the queries. It offers a much better way to externalize the queries. 

Using this notation is easy. In your Mule app, organize your queries in the src/main/resources folder in their own files as shown below.

![](/img/2021/externalize-queries-mule-1.jpg)

Once you organize your queries properly, injecting those queries is easy.

![](/img/2021/externalize-queries-mule-2.jpg)

This organization helps in keeping the XML configuration file readable and easily maintainable.

```
<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core"
	xmlns:http="http://www.mulesoft.org/schema/mule/http"
	xmlns:db="http://www.mulesoft.org/schema/mule/db"
	xmlns="http://www.mulesoft.org/schema/mule/core"
	xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core 
		http://www.mulesoft.org/schema/mule/core/current/mule.xsd
		http://www.mulesoft.org/schema/mule/db 
		http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd
		http://www.mulesoft.org/schema/mule/http 
		http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
		http://www.mulesoft.org/schema/mule/ee/core 
		http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd">
	<flow name="demo-apiFlow"
		doc:id="e3653667-121e-47cf-b2a6-33345b66e2e4">
		<http:listener doc:name="/Patient" 
			doc:id="39e23081-26d6-4469-953d-0efd3219b34f" 
			config-ref="HTTP_Listener_config" 
			path="/Patient"/>
		<db:select 
			doc:name="Select all patients" 
			doc:id="b7c2a60b-d6bb-431b-bb27-10419910aae6" 
			config-ref="Database_Config">
			<db:sql ><![CDATA[${file::sql/select-patients-all.sql}]]></db:sql>
		</db:select>
		<ee:transform 
			doc:name="Prepare response" 
			doc:id="e2774a96-2f54-4a41-acc2-e8f271b29f52" >
			<ee:message >
				<ee:set-payload resource="dw/patients-all-response.dwl" />
			</ee:message>
		</ee:transform>
	</flow>
</mule>
```
