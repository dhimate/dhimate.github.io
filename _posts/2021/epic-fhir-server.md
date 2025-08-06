---
title: "Using Epic's FHIR APIs in Mule 4"
date: 2021-01-23
tags:
 - MuleSoft
categories:
 - Programming
toc: true
description: "Using Epic's FHIR APIs in MuleSoft"
---
## Introduction
FHIR (Fast Healthcare Interoperability Resources) is a new and emerging standard for interoperability in healthcare IT. Unlike the ancient HL7v2 standard, FHIR is open, extensible, and composable. It supports modern web standards like JSON, XML, HTTP(!), and Turtle. It also defines different data exchange methods like RESTful API, messaging, services, documents, and persistent storage. Best of all the FHIR documentation is outstanding: There is detail documentation for every type of resource, with lots of examples. Enough detail for anyone to create a compliant implementation. All of the data types like dates and times, terminologies, identifiers, references, numeric values, extensions are documented in detail, including how to read and write them.  It's probably one of the best documented and well thought out APIs I have worked with.

Epic made their FHIR APIs available on [https://fhir.epic.com/](https://fhir.epic.com/). In this post, we will discuss using Epic's FHIR APIs in Mule 4

## Prerequisites 
You will need an Epic developer account. You can get one for free by signing up [here](https://fhir.epic.com/Developer/Index). I have used Anypoint Studio 7.7 for Mule app.

## Configuration
### Certificate setup for JWS
1. We will use SMART Backend Services approach described [here](https://fhir.epic.com/Documentation?docId=oauth2&section=BackendOAuth2Guide) which uses signed JWT to get the access token. 

2. Generate a new public-private key pair, and get the certificate(s) in appropriate format
{{< codecaption lang="shell" title="" >}}
# generate a new public-private key pair
openssl genrsa -out Anypoint_Mulesoft.pem 2048  

# export the public key to a base64 encoded X.509 certificate
openssl req -new -x509 -key Anypoint_Mulesoft.pem -out Anypoint_PublicKey.pem -subj '/CN=Anypoint' -days 365

# export the keystore in pkcs12
openssl pkcs12 -export -in Anypoint_PublicKey.pem -inkey Anypoint_Mulesoft.pem -out Anypoint_Keystore.p12
{{< /codecaption  >}}

### Create an app in Epic
1. Login to https://fhir.epic.com/ and click on 'Build Apps'
{{< figure src="/img/2021/epic-fhir-server/epic-build-app.png" alt="dwarf" width="600px" >}}

2. Click on 'Create' and enter your app details as follows. Make sure to select the Application Audience as 'Backend Systems'. For Sandbox JWT signing public key, upload the public key store (Anypoint_PublicKey.pem) file generated in the previous step. Click on save.
{{< figure src="/img/2021/epic-fhir-server/epic-create-app.png" alt="dwarf" width="600px" >}}


3. You should see the Client ID and Non-Production Client ID for your app. Select the SMART on FHIR version as R4, enter the summary,  accept the terms of usage and click on 'Save & Ready for Sandbox'. 
{{< figure src="/img/2021/epic-fhir-server/epic-app-overview.png" alt="dwarf" width="600px" >}}

*In my experience it usually takes a couple of hours for your app to be usable in Epic. It took me ~2 hours for my app to be usable*

## Mule application details
1. Create a new Mule application, and copy the pkcs12 formatted keystore (Anypoint_Keystore.p12) created earlier, in the `src/main/resources` folder.

2. To generate a signed JWT (JWS), we will use [Java JWT library](https://github.com/jwtk/jjwt). In the pom.xml of your Mule application, include the following dependencies. (I have included Apache log4j dependencies, these are optional).
{{< codecaption lang="xml" title="" >}}

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.14.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.0</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> 
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.65</version>
    <type>jar</type>
</dependency>

{{< /codecaption  >}}

3. In Mule application's `src/main/java` folder, create a new Java package `com.dhimate.demo`, class `JWTProvider` and use following code snippet to generate signed JWT.
{{< codecaption lang="java" title="" >}}

package com.dhimate.demo;

import io.jsonwebtoken.Jwts; 

import java.io.File;
import java.io.FileInputStream;
import java.net.URL;
import java.security.KeyStore;
import java.security.PrivateKey;
import java.util.Date;
import java.util.Enumeration;
import java.util.UUID;
import java.util.concurrent.TimeUnit;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;


public class JWTProvider {
//	private static final String PRIVATE_KEY_FILE_RSA = "Anypoint_Keystore.p12";
//	private static final String PRIVATE_KEY_PASSWORD = "CHANGEIT";
//	private static final String EPIC_CLIENT_ID = "86121d55-247e-4a0b-b3c0-6fa28c995f72";
//	private static final String EPIC_FHIR_URL = "https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token";
	
	private static final Logger logger = LogManager.getLogger("JWTProvider");
	public static String getToken(String PRIVATE_KEY_FILE_RSA, 
                                String PRIVATE_KEY_PASSWORD, 
                                String EPIC_CLIENT_ID, 
                                String EPIC_FHIR_URL) throws Exception {
		
		logger.info("Generating JWS token");
		
		String jws = Jwts
				.builder()
				.setHeaderParam("alg", "RS384")
				.setHeaderParam("typ", "JWT")
				.setIssuer(EPIC_CLIENT_ID)
				.setSubject(EPIC_CLIENT_ID)
				.setAudience(EPIC_FHIR_URL)
				.setId(UUID.randomUUID().toString())
				.setExpiration(new Date(System.currentTimeMillis() + TimeUnit.MINUTES.toMillis(5)))
				.signWith(getKey(PRIVATE_KEY_FILE_RSA, PRIVATE_KEY_PASSWORD))
				.compact();
		
		return (jws);
	}

	private static PrivateKey getKey(String PRIVATE_KEY_FILE_RSA, 
                                    String PRIVATE_KEY_PASSWORD) throws Exception {
		
		KeyStore keystore = KeyStore.getInstance("PKCS12");
		URL resource = JWTProvider.class.getClassLoader().getResource(PRIVATE_KEY_FILE_RSA);
		FileInputStream is = new FileInputStream(new File(resource.toURI()));
		
		keystore.load(is, PRIVATE_KEY_PASSWORD.toCharArray());
		Enumeration<String> aliases = keystore.aliases();
		String keyAlias = "";
		
		while (aliases.hasMoreElements()) {
			keyAlias = (String) aliases.nextElement();
		}

		PrivateKey key = (PrivateKey) keystore.getKey(keyAlias, PRIVATE_KEY_PASSWORD.toCharArray());
		return (key);
	}
}
 
{{< /codecaption  >}}

Your Mule application should look like this.

{{< figure src="/img/2021/epic-fhir-server/epic-mule-app-structure.png" alt="dwarf" width="300px" >}}

3. In Mule application's `src/main/java` folder, create a new Java package `com.dhimate.demo`, class `JWTProvider` and use the following code snippet to generate signed JWT.

4. Calling Epic's FHIR APIs is a two-step process. 
  - In the first step we need to get the access token using the signed JWT token
  - In the second step we will use the access token received in the step above to access FHIR resource
{{< figure src="/img/2021/epic-fhir-server/epic-mule-flow.png" alt="dwarf" width="600px" >}}

5. Let's look at the configuration of important components
  - Generate signed JWS token using `Invoke Static` component
    {{< figure src="/img/2021/epic-fhir-server/epic-mule-flow-generate-JWS.png" alt="dwarf" width="600px" >}}
  - Prepare request payload to get access token
    {{< figure src="/img/2021/epic-fhir-server/epic-mule-flow-prepare-token-request.png" alt="dwarf" width="600px" >}}
  - Get the access token
    {{< figure src="/img/2021/epic-fhir-server/epic-mule-flow-get-access-token.png" alt="dwarf" width="600px" >}}
  - Call FHIR API to get the data
    {{< figure src="/img/2021/epic-fhir-server/epic-mule-flow-epic-fhir-api.png" alt="dwarf" width="600px" >}}

6. Once this is done, we can add a simple HTTP listener and connect our flow to it to get the data from Epic's FHIR API. Epic has provided [test data](https://fhir.epic.com/Documentation?docId=testpatients) in the sandbox. Let's use it to search for a patient record.
{{< figure src="/img/2021/epic-fhir-server/epic-fhir-postman.png" alt="dwarf" width="600px" >}}