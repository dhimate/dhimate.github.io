---
title: "Understanding SMART on FHIR"
date: 2021-08-12
categories:
 - Programming
description: "Easy to understand explanation of SMART on FHIR"
---
Today practitioners and patients want to use innovative healthcare apps. Oftentimes they have their favorite app that they already use. To provide the best practitioner and patient experience, these apps need access to patient’s personal health information like allergies, or medications. This type of clinical information is stored inside the electronic health record systems (EHRs). Ultimately the apps are dependent on backend systems like EHRs to power their functionality. 

There are more than 50 EHR systems popular in the US alone. For an healthcare application developer this would be a nightmare, if he needs to build his app to work with 50 different EHR systems. 

Thanks to industry standards like FHIR and SMART on FHIR, this problem is easily solved. 

FHIR provides a standardized data model to represent healthcare data, whereas the SMART on FHIR provides a consistent approach to security and data access requirements. 

In this post let’s see some basics of SMART on FHIR. 

### What is SMART on FHIR ###

SMART stands for _**S**ubstitutable **M**edical **A**pplications and **R**eusable **T**echnology_. 

SMART app is a web or mobile app, embedded in an Electronic Health Record (EHR) system or standalone. It's substitutable, which means if you don’t like a particular app, you can choose another one or substitute it with something else. The app is reusable and should work with any EHR or backend system.

Simply put, SMART on FHIR provides a workflow to securely request access to data, as well as receive and use the data. And ultimately enables Electronic Health Record systems to become more like platforms. 

SMART on FHIR addresses these key concerns

- Identity and access management
- Access to the data
- Launch framework
- Scopes and launch context


Let’s look into the details of each of these for a bit.

### Identity and access management ###
SMART relies on the OIDC - OpenID Connect identity management protocol. 

Built on top of OAuth 2.0, it is essentially a flavor or a profile of OIDC, customized for use in healthcare applications.It defines a method through which an app requests authorization to access a FHIR resource, and then uses that authorization to retrieve the resource.

### Access to the data ###

SMART on FHIR uses FHIR to access the data. 

FHIR provides a standardized representation of healthcare data, irrespective of how it is stored or where it is coming from. Using a standardized representation of the data makes these apps substitutable as well as reusable.

### Launch framework ###
SMART on FHIR provides a framework for four different use cases 
- Patient apps that launch standalone
- Patient apps that launch from an EHR portal
- Provider apps that launch standalone
- Provider apps that launch from an EHR portal

### Scopes and launch context ###
SMART provides three different categories of scopes for different kinds of data.  Launch context is a negotiation where client asks for specific launch context parameters, and server decides which launch context parameters to provide using client’s request as an input. 

- Clinical data

    SMART on FHIR defines read and write permissions for patient specific and user level access. 
    
    E.g. 
    - `patient/Observation.read `
    - `user/Appointment.read`
    - `patient/*.*`

- Contextual data

    Apps typically rely on contextual data like the currently open patient record in an EHR. They explicitly request the EHR context by using scopes like
    
    - `launch=xyz`
    - `launch/patient`
    - `launch/encounter`

- Identity data 
    
    Some apps need to authenticate the clinical end-user. They use following openid connect scopes to request the identity information.
    - `openid`
    - `fhirUser`


This concludes my brief introduction to SMART on FHIR. 

SMART on FHIR provides an open, free and standards based API. Developers can use to write an app once and have it run anywhere in healthcare IT system.



