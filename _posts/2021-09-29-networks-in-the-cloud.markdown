---
publish: false # This file is a template, remove this line to publish a new post created from it
layout: post
title:  "Exercise 6 - Web Apps in the cloud"
date:   2021-09-24 22:39
tags: Azure, Cosmos DB, Azure Functions, Serverless
---

<h1 style="color:red;">WIP, not complete!</h1>


## Preface
This session's assignment was to argument for ...

### The scenario:
I work at a company which has an internal application they company would like to modernize. The app runs on an on-premises server and uses internal resources, and it deals with classified data which cannot be sent over the **public** internet. Because of the nature of the app the CTO of the company is reluctant to the use of a PaaS solution when modernizing the app.

An important step in the modernization of the app is the implementation of an enterprise bus, and the development team's preferred solution is Azure Service Bus. The CTO as mentioned is opposed to the idea since data cannot be sent over the internet, and needs to be convinced to change their mind.

The goal of this post is to explain certain concepts and services to the CTO (such as **Virtual Private Cloud**), and present sound arguments as to why it would be possible to use **Azure Service Bus** through the use of an **Azure Private Link**.

If the CTO can be convinced, it would for the company open up the possibility of using other Azure PaaS services which would make the life easier for the developers which is beneficial for the company. Possible improvements are higher efficiency, better security, cheaper operation, and less lead time when developing new solutions.

The CTO has the following qualities:
- Not hard to convince, as long as the arguments are factually correct.
- Visual: loves diagrams and drawings.
- Technically inclined: will understand technical terms.

~~
Ni jobbar i ett företag som har en intern applikation som ni gärna vill modernisera, denna interna applikation använder tillgång till interna server och resurser, och jobbar med klassificerade data som inte får öppet skickas via nätat, och därför är det i följa din CTO inte möjligt att använda en PaaS moln-lösning.

Ett viktigt punkt i modernisering är att implementera en enterprise bus, och ni vill gärna använda er av Azure Service Bus, men tyvär säger eran CTO säger “nej”, på grund av data inte får skickas öppet.

Skriv (i eran blogg) ett argument till eran CTO som förklara och övertyga hen om att ni med hjälp av ett Azure Private Link kan använda Azure Service Bus i eran interna applikation. Förklara begrepp som Virtual Private Cloud i eran förklaring.

Om ni lyckas att övertyga eran CTO till att tillåta att ni använder Azure Service Bus, kommer ni i framtiden att kunna använda andra Azure PaaS tjänster (vilket gör livet som utvecklare mycket bättre) :-)

Fakta om eran CTO, hen är:

    inte så svår att övertyga, om bara ni använder fakta
    visuell och älskar därför diagram och ritningar (men se till att göra egna, använd t.ex. draw.io)
    fan av teknisk, så var inte rädd om att använda tekniska termer
~~

Hints:
[Connect an on-premises network to a Microsoft Azure virtual network](https://docs.microsoft.com/en-us/microsoft-365/enterprise/connect-an-on-premises-network-to-a-microsoft-azure-virtual-network?view=o365-worldwide)

## Dear boss person

## Arguments for 


### What is Azure Service Bus

**Azure Service Bus** is a very robust and mature technology. Migrating to 


### What is Azure Private Link

With **Azure Private Link** we can create a secure private endpoint (**Azure Private Endpoint**) within our **VPN** through which we will have private and secure communication with our customer's services, Azure **PaaS (Platform as a Service)** services and even 3rd party services. All traffic travels on Microsoft's backbone network. This means we do not send any data over, or expose our services to, the public internet.

Within any configuration we can 

We can create our own **Azure Private Service** within our virtual network to provide means for our customers to open private connections to our services. This requires the service to be behind a load balancer.


### What is a Virtual Private Cloud?

A **VPC** (**Virtual Private Cloud**) is a **private cloud** hosted remotely (in the **public cloud**). It is however isolated from the rest of the public cloud and therefor secure. **VPC** combines the benefits of private cloud computing (isolation, security, availability) with all of the benefits of using the **public cloud** such as scaling, redundancy and reliability. But how can a VPC be isolated from the rest of the public cloud it is hosted in? One of the main technologies behind this is **VPN** (**Virtual Private Network**). A **VPN** can send data between on-premise resources and off-premise resources in a number of ways, but communication is always encrypted, this means that the data is scrambled while in transit between resources. There are two options for connecting on-premise devices with a **VPN** over the internet. These are **point-to-site VPNs** and **site-to-site VPNs**. With Point-to-site **VPN** you can connect a single device to a virtual network


I do understand your reservations about moving us to the cloud and no solution is ever going to be 100% secure, but these are well established technologies, with very large user bases who can rely on them being secure and reliable.
Moving our solution to the cloud with the help of Azure's **PaaS (Platform as a Service)** will the vast resources of Azure cloud computing available to us. In the long term this would mean easier and faster expansion of our business. Short term it would make life easier for our developers, with less to resources being spent on operations and more on research & development. We would be able to iterate faster, and reduce lead time to market. We would be able to offer more options for our customers.



## Sources & Links
- [Url Title][url-id]


[url-id]: https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview
[url-id]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction
