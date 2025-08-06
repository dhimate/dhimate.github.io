---
title: "How to Use Netcat to Debug Cloudhub Networking Issues"
date: 2021-01-02
tags:
 - MuleSoft
categories:
 - Programming
---
This is a simple trick I use to debug connection issues with the mule app deployed to CloudHub.

First of all its important to understand that there are different ways to set up your application

**1. Using shared load balancer (i.e. cloudhub.io domain) with HTTP protocol.**

In this scenario, the client application makes a request to a shared load balancer URL using **HTTP** protocol. The SLB forwards the request to the upstream mule application listening on port 8081.

Note that in this scenario the SLB URL *receives* the request on port 80, and forwards it to mule application listening on port 8081.
{{<mermaid >}}
flowchart LR
   l1[Shared Load Balancer]-.->c1 
    subgraph A[Shared Load Balancer w/ HTTP]
        u(client)--http-->c1
        c1(hello-world.cloudhub.io<br/>)--port 8081-->a1
        subgraph VPC 10.0.0.0/24
        a1(mule-worker-hello-world.cloudhub.io:8081)--->id1((hello-world))
      end
    end
 style A fill:#FFF,stroke:#DDD,stroke-width:2px
{{</mermaid >}}

**2. Using shared load balancer (i.e. cloudhub.io domain) with HTTPS protocol.**

In this scenario, the client application makes a request to a shared load balancer URL using **HTTPS** protocol. The SLB forwards the request to the upstream mule application listening on port 8082.

Note that in this scenario the SLB URL *receives* the request on port 443, and forwards it to mule application listening on port 8082.
{{<mermaid >}}
flowchart LR
   l1[Shared Load Balancer]-.->c1 
    subgraph B[Shared Load Balancer w/ HTTPS]
        u(client)--https-->c1
        c1(hello-world.cloudhub.io<br/>)--port 8082-->a1
        subgraph VPC 10.0.0.0/24
        a1(mule-worker-hello-world.cloudhub.io:8082)--->id1((hello-world))
      end
    end
 style B fill:#FFF,stroke:#DDD,stroke-width:2px
{{</mermaid >}}

**3. Using dedicated load balancer (i.e. vanity domain) with HTTPS protocol.**

In this scenario, the client application makes a request to a dedicated load balancer URL(exposed by a vanity domain) using **HTTPS** protocol. The DLB forwards the request to the upstream mule application listening on port 8091.

Note that in this scenario the DLB URL *receives* the request on port 443, and forwards it to mule application listening on port 8091.
{{<mermaid >}}
flowchart LR
   l1[Dedicated Load Balancer<br/>Vanity Domain]-.->c1 
    subgraph C[Dedicated Load Balancer w/ upstream HTTP]
        u(client)--https-->c1
        c1(api.example.com<br/>)--port 8091-->a1
        subgraph VPC 10.0.0.0/24
        a1(mule-worker-hello-world.cloudhub.io:8091)-->id1((hello-world))
        end
    end
 style C fill:#FFF,stroke:#DDD,stroke-width:2px,font-face:Ubuntu
{{</mermaid >}}

**4. Using dedicated load balancer (i.e. vanity domain) with HTTPS protocol.**

In this scenario, the client application makes a request to a dedicated load balancer URL(exposed by a vanity domain) using **HTTPS** protocol. The DLB forwards the request to the upstream mule application listening on port 8092.

Note that in this scenario the DLB URL *receives* the request on port 443, and forwards it to mule application listening on port 8092.
{{<mermaid >}}
flowchart LR
   l1[Dedicated Load Balancer<br/>Vanity Domain]-.->c1
    subgraph C[Dedicated Load Balancer w/ upstream HTTPS]
        u(client)--https-->c1
        c1(api.example.com<br/>)--port 8092-->a1
        subgraph VPC 10.0.0.0/24
        a1(mule-worker-hello-world.cloudhub.io:8092)-->id1((hello-world))
        end
    end
 style C fill:#FFF,stroke:#DDD,stroke-width:2px,font-face:Ubuntu
{{</mermaid >}}

**5. Not using load balancers.**

This is one of those scenarios where you are not using Anypoint Platform load balancers. You are either using third-party load balancers, or your applications are using non HTTP protocols like a simple socket, or MLLP. In this case, the client application makes a request directly to the application (mule-worker). 
{{<mermaid >}}
flowchart LR
        u(client)--HTTPS-->a1
        subgraph VPC 10.0.0.0/24
        a1(mule-worker-hello-world.cloudhub.io:8092)-->id1((hello-world))
        end
{{</mermaid >}}

As seen above, to make your application endpoint available to the clients

- When using a shared load balancer, your application should be listening on an appropriate port (8081 or 8082)
- When using a dedicated load balancer, your application should be listening on an appropriate port (8091 or 8092), and the load balancer's upstream HTTP protocol should be configured properly.
- The VPC firewall should be configured properly to allow the ingress traffic on the appropriate port.

If there is a misconfiguration, your application endpoint will not be accessible. Your clients will get errors like 'Bad Gateway', or 'Timeout'. The easiest way to debug these types of errors is to use netcat to perform port scanning and identify open ports on your application.

For example, to check if your application `hello-httpbin` is listening on port 8081 you can use
```shell

  nc -w 1 -G 1 -z -v mule-worker-hello-httpbin.cloudhub.io 8081

  Connection to mule-worker-hello-httpbin.cloudhub.io port 8081 [tcp/sunproxyadmin] succeeded!

```
In this you can see, the application name is prefixed with `mule-worker`, to get the correct DNS name. For more details about this convention, please see the DNS records section in [Cloudhub Networking Guide](https://docs.mulesoft.com/runtime-manager/cloudhub-networking-guide#dns-records). The other options of netcat (or nc) are as follows
 - -w: Timeout when a connection is idle for more than specified seconds. 
 - -G: TCP connection timeout in seconds.
 - -z: Just scan for listening daemons, without sending any data to them.
 - -v: Give more verbose output.

If you are not sure about the specific open port in your application, you can specify the range and identify all open ports using

```shell

  nc -w 1 -G 1 -z -v mule-worker-hello-httpbin.cloudhub.io 8081-8092
  Connection to mule-worker-hello-httpbin.cloudhub.io port 8081 [tcp/sunproxyadmin] succeeded!
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8082 (tcp) failed: Connection refused
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8083 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8084 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8085 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8086 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8087 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8088 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8089 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8090 (tcp) failed: Operation timed out
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8091 (tcp) failed: Connection refused
  nc: connectx to mule-worker-hello-httpbin.cloudhub.io port 8092 (tcp) failed: Connection refused

```
In this, you can see that the application only has port 8081 open. Also interesting to note that the ports 8082, 8091, and 8092 are 'refusing connection', as they are allowed in the firewall but they are not exposed in the application. Whereas for the remaining ports we are getting an 'Operation timed out' error as they are blocked at the firewall.

The simple netcat/nc utility has saved me a lot of time. I hope it helps you as well.
