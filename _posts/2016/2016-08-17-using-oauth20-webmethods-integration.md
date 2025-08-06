---
title: "Using OAuth 2.0 with webMethods Integration Server"
date: 2016-08-17
layout: post
tags:
 - webMethods
categories:
 - Programming 
toc: true
---

webMethods Integration Server supports OAuth 2.0. The Integration Server (IS) can be used as an OAuth client, an authorization server, or a resource server. This post describes how to use OAuth 2.0 with Integration Server in a simplified format.

For ease of understanding, we will consider IS as an authorization server and resource server in this post. However, the same concept can be used to set up any role that you may want IS to perform in your architecture.

Integration Server supports two types of OAuth grant types

**Authorization Code**:
This is a secure way of obtaining an OAuth token, where the client needs to authorize himself with the authorization server. The authorization server responds with the authorization code, which the client uses to obtain the access token. This approach is typically used by clients hosted on web server-based applications where the application code runs on the server. 

**Implicit**: 
This approach is used by mobile apps or browser-based applications. In this approach, the client does not need to authorize himself with the authorization server. The access token is passed through the browser, as a fragment in the URL. A simple javascript code can be used to extract the access token from the fragment. 

We will see how to implement both approaches in the integration server. For both approaches, you need to do some basic setup on the IS console.

- **Client Configuration**:

A client must register himself and obtain client_id from the integration server.  In the IS console go to Security->OAuth->Client Registration page and click on 'Register Client'. Enter the information as shown below.
Redirect URL should be the URL, where IS will redirect the page after authorizing the request. I have created a simple RESTful service on the IS as a redirect URL.


{{< figure src="/img/2016/OAuth_1.PNG" alt="dwarf" width="700px" >}}


Remember that the Client ID will be generated when you click Save Changes.

{{< figure src="/img/2016/OAuth_2.PNG" alt="dwarf" width="700px" >}}

Your client will use this 'Client ID' to authorize himself and request an access token.

- **Scope Management**:

The scope indicates the resources the client can access on behalf of resource owner. You need to indicate one or more folders or services on the IS in the scope. Once a client is granted access to a scope, he can access the folders and services included in that scope.

Go to Security->Oauth->Scope Management page and click on 'Add Scope'. Once the page is open, mention the scope details as shown below

Here we have included one service 'OAuthDemo' in the scope. This is an URL Alias for /rest/Sandbox.OAuthDemo folder on the server

{{< figure src="/img/2016/OAuth_3.PNG" alt="dwarf" width="700px" >}}



Once the scope is added, map the scope with the client using the 'Associate Scope to Clients' option on the 'Scope Management' page.

{{< figure src="/img/2016/OAuth_4.PNG" alt="dwarf" width="700px" >}}


Once this setup is done your client can request the access token either for the Authorization Code approach or Implicit approach.

- Authorization Code:

Requesting token for authorization code approach is a 2 step process. In the first step, the client authorizes himself with the authorization server using `pub.auth:authorize` service and receives the authorization code. Once the authorization code is received, it can be used to obtain the access token using `pub.auth:getAccessToken` service.


When the client initiates the `pub.oauth:authorize` request, it brings up a page for the resource owner to either approve the access request or reject it. 

{{< figure src="/img/2016/OAuth_5.PNG" alt="dwarf" width="700px" >}}

When the resource owner approves the request, the integration server generates the authorization code and redirects the page to the 'redirect URL' specified in the client configuration.  

The service hosted at redirect URL passes the authorization code to `pub.oauth:getAccessToken` service to exchange authorization code for an access token as shown below

{{< figure src="/img/2016/OAuth_6.PNG" alt="dwarf" width="700px" >}}

The client application can now use the access token to access the resource described in the Scope.

- Implicit:

Requesting token for implicit approach is one-step process. 
When the client initiates the `pub.oauth:authorize` request, it indicates the implicit approach by mentioning the response type as 'token' in the input to `pub.oauth:authorize` service. The service brings up a page for resource owner to either approve the access request or reject it.  

{{< figure src="/img/2016/OAuth_7.PNG" alt="dwarf" width="700px" >}}

When the resource owner approves the request the integration server generates the access token and includes it as a fragment in the redirect URI, which the client application can extract it using simple javascript code. 

{{< figure src="/img/2016/OAuth_8.PNG" alt="dwarf" width="700px" >}}
 
 Integration Server administrator can verify all the tokens in the IS console

{{< figure src="/img/2016/OAuth_9.PNG" alt="dwarf" width="700px" >}}


The client application can use the token as a bearer token to access the resource on the server. 

{{< figure src="/img/2016/OAuth_10.PNG" alt="dwarf" width="700px" >}}


If the client application tries to use an invalid token or tries to access a service which is not in the scope, it will get an error

{{< figure src="/img/2016/OAuth_11.PNG" alt="dwarf" width="700px" >}}






