---
title: "Integration Server Class Loading for Custom Jars"
date: 2016-08-16
layout: post
tags:
 - webMethods
categories:
 - Programming 
toc: true
---
If a specific functionality is not available out of box from the integration server, you often need to use external jars or java libraries in IS. e.g. If you want to generate a PDF document or excel spreadsheet on your IS, you need to use your preferred libraries like IText or Apache POI. 

It is not recommended to place your custom jars in the IntegrationServer/lib/jars folder. But Integration Server provides you couple of places to place these jars depending on how you want them to be loaded. 

1. packages/<package name>/code/jars or packages/<package name>/code/classes:  The jar file placed in these folders are only accessible to java services inside the same package and the packages dependent on this package. If there is any change in the jar, simply reloading of the package is sufficient to have the classes loaded in memory. 

2. packages/<package name>/code/jars/static are accessible to all java services across the entire IS. If there is any change in the jar, a restart of IS is required to activate any additions or changes to these.

