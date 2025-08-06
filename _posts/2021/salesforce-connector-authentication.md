---
title: "How to Use Different Authentication Options for Salesforce Connector in Mule 4"
date: 2021-01-16
description: "Authentication options for Salesforce Connector in Mule 4"
tags:
 - MuleSoft
categories:
 - Programming
toc: true
---
## Introduction
MuleSoft offers multiple connectors for Salesforce integration. This post is about Salesforce Connector for Mule 4. This connector helps in integrating with Sales Cloud, Service Cloud, Salesforce Platform, and Force.com. We will specifically look into different options for authentication with Salesforce.

## Software Prerequisites 
You need a Salesforce developer account. You can get one for free by signing up [here](https://developer.salesforce.com/signup). I have used the following versions for MuleSoft software.
1. Anypoint Studio 7.7
2. Salesforce Connector 10.8.0
## Configuration
### Security Token in Salesforce
Salesforce security token is an alphanumeric code associated with your password. You do not need a security token if you are trying to authenticate from an IP address that is inside your Salesforce org's trusted IP range, or your Salesforce profile's login IP range. When you sign up for a new developer account, your Salesforce org does not have trusted IP ranges. You can either set up trusted or login IP ranges in your org, or use the security token for authentication. Generating security token is simple. 
1. Go to settings  
{{< figure src="/img/2021/salesforce-connector-auth/settings-salesforce.jpg" alt="dwarf" width="300px" >}}

2. Reset security token by using the 'Reset My Security Token' menu and clicking on the 'Reset Security Token' button. 
{{< figure src="/img/2021/salesforce-connector-auth/reset-security-token.jpg" alt="dwarf" width="500px" >}}

3. You should receive your new security token in your email account that you used to sign up for the Salesforce developer account. 
### Certificate setup in Salesforce
For certificate-based security, you need to set up a certificate in Salesforce.
1. Go to setup 
{{< figure src="/img/2021/salesforce-connector-auth/setup-salesforce.jpg" alt="dwarf" width="300px" >}}

2. Search for certificate and key management and select the option
{{< figure src="/img/2021/salesforce-connector-auth/certificate-key-management-menu.jpg" alt="dwarf" width="300px" >}}

3. Click on 'Create Self-Signed Certificate' button to create a new certificate and enter the information and save it.
{{< figure src="/img/2021/salesforce-connector-auth/create-self-signed-certificate.jpg" alt="dwarf" width="800px" >}}

4. Once certificate is created, download the certificate (.crt) file. We need it for setting up the connected app.
{{< figure src="/img/2021/salesforce-connector-auth/download-certificate.jpg" alt="dwarf" width="800px" >}}

5. Export the certificate to keystore. We need it to configure the connection information in Mule app.
{{< figure src="/img/2021/salesforce-connector-auth/export-to-keystore.jpg" alt="dwarf" width="800px" >}}

6. Enter the password when exporting to keystore
{{< figure src="/img/2021/salesforce-connector-auth/enter-password.jpg" alt="dwarf" width="800px" >}}

7. Make sure you save both the .crt file (downloaded in step 4 above) and the .jks file safely on your disk.

### Connected App setup in Salesforce
1. Go to setup 
{{< figure src="/img/2021/salesforce-connector-auth/setup-salesforce.jpg" alt="dwarf" width="300px" >}}

2. Search for app manager and select the 'App Manager' option
{{< figure src="/img/2021/salesforce-connector-auth/app-manager-option.jpg" alt="dwarf" width="300px" >}}

3. Use the 'New Connected App' button to create a new connected app.
{{< figure src="/img/2021/salesforce-connector-auth/new-connected-app.jpg" alt="dwarf" width="800px" >}}

4. Fill in name and email. Select ‘Enable OAuth Settings’ Enter two Callback URLs 

    - https://oauthdebugger.com/debug
    - http://localhost:8081
    
Select ‘Use digital signatures’ and upload the .crt file received when setting up the certificate above.
Select all the OAuth Scopes if possible
{{< figure src="/img/2021/salesforce-connector-auth/create-connected-app.jpg" alt="dwarf" width="800px" >}}
Note the client_id and client_Secret

5. In your connected app's page, click on Manage button to review and update the app policies. Click on Manage.
{{< figure src="/img/2021/salesforce-connector-auth/manage-connected-app-menu.jpg" alt="dwarf" width="800px" >}}

6. And then on 'Edit Policies'
{{< figure src="/img/2021/salesforce-connector-auth/edit-policies.jpg" alt="dwarf" width="800px" >}}

7. In the Permitted Users option you can see two options
      
    - Admin approved users are pre-authorized.

      We will not go into the details of this option, but if you want to use it you can use the following steps        
      - Go to Settings > Manage Users > Profiles
      - Edit the profile associated with the user
      - For ‘Connected App Access’ under the profile, select the checkbox against the connected app. Save.

    - All users may self-authorize.

      We will use this option. Ensure that you select this option and save the policy.
      {{< figure src="/img/2021/salesforce-connector-auth/connected-app-policy.jpg" alt="dwarf" width="800px" >}}
8. Open your browser and go to https://oauthdebugger.com/. Enter following details

   - Authorization url = ‘https://login.salesforce.com/services/oauth2/authorize’
   - Redirect URI = ‘https://oauthdebugger.com/debug’
   - Client ID = obtained from the connected app
   - Scope = ‘api refresh_token offline_access’ 
   - Response type = ‘code’
   - Response mode = ‘query’
   - Click on “send request”

{{< figure src="/img/2021/salesforce-connector-auth/oauth-debugger-screen.jpg" alt="dwarf" width="800px" >}}   

9. It should redirect you to enter your credentials on Salesforce login screen and you should receive an authorization code. 
{{< figure src="/img/2021/salesforce-connector-auth/salesforce-oauth-consent.jpg" alt="dwarf" width="500px" >}}

10. This will complete the connected app setup.

## Basic Authentication
Basic authentication is the easiest way to authenticate with Salesforce. For basic authentication you need username and password, and optionally a [security token](#configuration).
{{< figure src="/img/2021/salesforce-connector-auth/basic-auth-test-connection.jpg" alt="dwarf" width="800px" >}}   

## OAuth JWT
For OAuth JWT connection copy and paste the key store file (.jks) that you created in the certificate setup section into the src/main/resources folder. 

- Consumer Key – Consumer key of the Connected App that was created in Salesforce
- Key Store – Keystore file (.jks) that you downloaded from the certificate setup
- Store Password – Keystore password that was created when you generated the key store file
- Principal – Salesforce username of the user that was approved against the authorization URL

{{< figure src="/img/2021/salesforce-connector-auth/oauth-jwt-connection.jpg" alt="dwarf" width="800px" >}}


## OAuth SAML
For OAuth SAML connection copy and paste the key store file (.jks) that you created in the certificate setup section into the src/main/resources folder. 

- Consumer Key – Consumer key of the Connected App that was created in Salesforce
- Key Store – Keystore file (.jks) that you downloaded from the certificate setup
- Store Password – Keystore password that was created when you generated the key store file
- Principal – Salesforce username of the user that was approved against the authorization URL

{{< figure src="/img/2021/salesforce-connector-auth/oauth-saml-connection.jpg" alt="dwarf" width="800px" >}}


## OAuth Username Password
For OAuth Username Password configure the following properties

- Consumer Key – Consumer key of the Connected App that was created in Salesforce
- Consumer Secret – Consumer secret of the Connected App that was created in Salesforce
- Username – Salesforce login id
- Password – Salesforce password
- Security Token - Security token associated with the username

{{< figure src="/img/2021/salesforce-connector-auth/oauth-username-password.jpg" alt="dwarf" width="800px" >}}