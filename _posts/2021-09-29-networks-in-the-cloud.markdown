---
layout: post
title:  "Exercise 7 - Cloud Networks"
date:   2021-09-29 22:59
tags: Azure, Azure Virtual Network, Virtual Private Network, Azure PaaS, VPN, VPC, VNet, Azure Private Link, ExpressRoute, Cloud Networking
---

## Preface

This session's assignment was convince the CTO of a company who is reluctant to move the company's app from the on-premises server and resources to an Azure PaaS solution.

### The (fictitious) scenario:
I work at a company which has an internal application they company would like to modernize. The app runs on an on-premises server and uses internal resources, and it deals with classified data which cannot be sent over the **public** internet. Because of the nature of the app the CTO of the company is reluctant to the use of a PaaS solution when modernizing the app.

An important step in the modernization of the app is the implementation of an enterprise bus, and the development team's preferred solution is Azure Service Bus. The CTO as mentioned is opposed to the idea since data cannot be sent over the internet, and needs to be convinced to change their mind.

The goal of this post is to explain certain concepts and services to the CTO (such as **Virtual Private Cloud**), and present sound arguments as to why it would be possible to use **Azure Service Bus** through the use of an **Azure Private Link**.

If the CTO can be convinced, it would for the company open up the possibility of using other Azure PaaS services which would make the life easier for the developers which is beneficial for the company. Possible improvements are higher efficiency, better security, cheaper operation, and less lead time when developing new solutions.

The CTO has the following qualities:
- Not hard to convince, as long as the arguments are factually correct.
- Visual: loves diagrams and drawings.
- Technically inclined: will understand technical terms.


Hints:
[Connect an on-premises network to a Microsoft Azure virtual network](https://docs.microsoft.com/en-us/microsoft-365/enterprise/connect-an-on-premises-network-to-a-microsoft-azure-virtual-network?view=o365-worldwide)

## Dear boss person

As you know, we would like to connect our internal app to an **Azure Service Bus** in our effort to modernize our app.
To securely send data between our on-premises app and the **Azure Service Bus** we would utilize a service called **Azure Private Link**, more on this later.

By taking this first step we can further down the line migrate more of our operations to Azure's PaaS offerings.

I would like to take a few minute of your time and explain some of the concepts and technologies we would like to use in an effort to reassure you that what we are suggesting will in fact keep our data secure.


### What is Azure Service Bus

**Azure Service Bus** is a message broker which takes care of things such as load balancing and safe routing and transferring of data between apps and services. You can think of it as a channel for messages where we can make sure the message stays until at least one app or service has processed the data. This means our apps and service connected to the service bus can be decoupled, i.e. they can be independent of other apps and services and completely autonomous. Decoupled apps means improved app reliability and scalability, meaning we can easily spread out the load to multiple instances if traffic to a specific app or service spikes.

Within **Azure Service Bus** you will find a concept called **topics and subscriptions**. In short this allows multiple apps/services (**consumers**) to process multiple messages at the same time, improving throughput, availability and scalability. This effectively balances the workload.

With **Azure Service Bus** we can also enable transactions, a way of grouping together multiple operations, making sure that all operations are successful. If one operation fails, the entire transaction fails, and vice versa. As an example, let's say a customer places an order in our store. To fullfil this order we need to; tell our warehouse to put together and send the items, update our database and send an invoice to the customer. If the invoice cannot for whatever reason be created or sent to the customer we do not want to send them their order or update our database. The entire order should fail. This is called an **atomic transaction**.

Messages in the bus can either be sent and received from a **message queue** or it can use **topics and subscriptions** as described above. With a **queue** messages are stored in the system until they are consumed by the app/service they are intended for. **Queues** are better suited for when there is a single intended receiver for a message.

**Azure Service Bus** is a very robust and mature technology. There are simple migrating paths we could use to migrate our app to Azure Service Bus.

![](/Molnapplikationer-Blogg/data/images/exercise-7-cloud-networks/diagram-azure-service-bus.png)


### What is a Virtual Private Cloud?

A **virtual private cloud** or a **VNet (Virtual Network)** is an isolated and highly secure computing environment for running VMs (Virtual Machines), apps and services.

Azure's **VPC** (**virtual private cloud**) offering is called **VNet** (**Azure Virtual Network**), which is a **private cloud** hosted remotely by Azure (in shared **public cloud** infrastructure). It is however isolated from the other tenant on the public cloud and we can control what goes in and what goes out. It is thus effectively a secure and private network. 

**VPC** combines the benefits of private cloud computing (isolation, security, availability) with  the benefits of using the **public cloud** such as scaling, redundancy and reliability.

By extending our on-premises network with a **VPC** we can for example host our app in Azure which can access data stored in a database on our premises, or vice versa. Or we can move everything to Azure, reducing operating complexity and costs.


### VPNs (Virtual Private Network) and Azure ExpressRoute

To send send data between on-premise resources and off-premise resources (**VNet**) in a number of ways, either encrypted over the internet or over a private ExpressRoute connection. If the data is encrypted it means means that the data is scrambled while in transit between resources.

#### What is VPN (Virtual Private Network)?

A VPN is a private secure tunnel between a device and a network or between two devices.
 There are two options for connecting on-premise devices with a **VPN** over the internet. These are **point-to-site VPNs** and **site-to-site VPNs**. With Point-to-site **VPN** you can connect a single device to a virtual network and with **site-to-site** you can connect two networks together.

#### What is Azure ExpressRoute?

**Azure ExpressRoute** lets us connect our on-promise network with our **VNet** over a private connection. It uses Microsoft's backbone network (**Azure global network**). This backbone network is a fast, reliable and secure private network created and owned my Microsoft and it does not send data over the internet. Data can be encrypted when sent over **ExpressRoute** but is not by default. Encrypting the transfer is less important because of the private nature of the connection.


### What is Azure Private Link

To connect our **VNet** (**Virtual Network**) to and access **Azure Service Bus** or any other of Azure's services, or even customer/partner services hosted in Azure, we can use something called **Azure Private Link**.

**Azure Private Link** provides private connectivity between VPCs, Azure services and our on-premises networks without exposing our traffic to the public internet.

**Azure Private Link** is a private and secure connection which uses Microsoft's backbone network (**Azure global network**). This backbone network is a fast, reliable and secure private network created and owned my Microsoft and it does not send data over the internet.

To create the connection we create secure private endpoints (**Azure Private Endpoint**), these are mapped to a specific isolated instance of a PaaS resource which means any unlikely intrusion does not get access to the entire service.

If we place our service behind an **Azure Load Balancer**, our customer can also utilize **Private Link** to directly access our services from their own **Virtual Network**.

![](/Molnapplikationer-Blogg/data/images/exercise-7-cloud-networks/diagram-vpc-private-link.png)


### Closing words

I do understand your reservations about moving us to the cloud and no solution is ever going to be 100% secure, but these are well established technologies, with very large user bases who need to rely on them being secure and reliable daily.
Moving our solution to the cloud with the help of Azure's **PaaS (Platform as a Service)** will the vast resources of Azure cloud computing available to us. In the long term this would mean easier and faster expansion of our business. Short term it would make life easier for our developers, with less to resources being spent on operations and more on research & development. We would be able to iterate faster, and reduce lead time to market. We would be able to offer more options for our customers, and we can extend our global reach.



## Sources & Links
- [Virtual Network - docs.microsoft.com](https://azure.microsoft.com/en-in/services/virtual-network/)
- [What is Azure Private Link? - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/private-link/private-link-overview)
- [What is Azure Private Link service? - docs.microsoft.com](https://docs.microsoft.com/da-dk/azure/private-link/private-link-service-overview)
- [Azure Express Route - docs.microsoft.com](https://docs.microsoft.com/en-ca/azure/expressroute/expressroute-introduction/)
- [What is Azure Service Bus? - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)
- [What is Azure Virtual Network? - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
- [What is Azure ExpressRoute? - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
- [What is VPN Gateway? - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways)
- [Service Bus queues, topics, and subscriptions - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions)
- [Azure architecture icons - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/architecture/icons/)
- [Microsoft global network - docs.microsoft.com](https://docs.microsoft.com/en-us/azure/networking/microsoft-global-network)
- [Virtual Private Cloud (VPC) - ibm.com](https://www.ibm.com/cloud/learn/vpc)
- [What is a virtual private cloud (VPC)? - cloudfare.com](https://www.cloudflare.com/en-gb/learning/cloud/what-is-a-virtual-private-cloud/)
