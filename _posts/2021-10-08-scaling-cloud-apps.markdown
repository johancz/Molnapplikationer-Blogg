---
publish: false # This file is a template, remove this line to publish a new post created from it
layout: post
title:  "Exercise 10 - Scaling Web Apps (up and out)"
date:   2021-10-08 19:59
tags: Azure, Scaling, Autoscale
---

<h1 style="color:red;">WIP, not complete!</h1>


## The assignment (in Swedish)

Förklara skildnad på att skala en applikation horisontalt och vertikalt
Hur påverker det kostnad att ha en applikation Azure som skalar horisontalt vs vertikalt?
    För en app service
    En en virtuell maskin
Inte alla Azure Service App plans gir möjlighet att skala, vilka gir vilka möjligheter?


## What is scaling?

Scaling is a configuring your infrastructure to respond to demand.


## What is the difference between horizontal and vertical scaling?

**Horizontal scaling**, also knows as in-and-out scaling, is when you scale the number of instances of your app/resource. This often means spooling up the app on a new server.

**Vertical scaling**, or up-and-down scaling, is when an instance's/server's computing power and computing resources are increases, or decreased, in response to current demand. This could mean increasing/decreasing RAM, core or virtual CPU count, or storage.

### What should I use?

Microsoft recommends having at least two instances running at all times. This improves your app's reliability, since if one instance goes down for whatever reason, the second instance can take responsibility for the traffic the first instance was supposed to handle. Autoscaling or any equivalent should then kick in based and spool up a new instance.

If reliability isn't a major concern, vertical scaling might be a better option, since it should be cheaper than scaling horizontally.

Now let's say if your company runs an e-commerce website, reliability should be a top priority and you would wan't to run at least two instances, and scale out as demand increases and scale back in when traffic decreases

Scaling horizontally doesn't improve performance of your site as such, and it can actually introduce a performance hit. Of course it depends on how your backend is setup. If scaling horizontally means spooling up a new server for each instance (including a database), you will need to sync your databases regularly, and this can introduce a hit to your app's performance.

Vertical Scaling can be used when you need to react quickly to fix a performance issue, or when you have smaller spikes in traffic to you app.

Horizontal scaling is perhaps better left to when you have hit the roof of what vertical scaling can accomplish. After all, all computers (servers) have limited computing power.


## How does horizontal vs vertical scaling affect cost of an app hosted in Azure

I wasn't expecting there being no price difference between horizontal and vertical scaling on Azure, for both **App Service** and **VM**.

![](/Molnapplikationer-Blogg/data/images/exercise-10-scaling/assignment-10-azure-pricing-app-service-and-VM-scaling-vertical-and-horizontal.png)

These are common variables for all data entries:
- Region: North Europe
- Operating System: Windows
- Prices are for 1 month.

**First I compared **App Service** prices using the "Standard" tier.**

The **instance** used by both horizontal and vertical scaling with x1 scaling was "S1: 1 Core(s), 1.75 GB RAM, 50 GB Storage".
For vertical scaling, I changed the **instance** to "S2: 2 Core(s), 3.5 GB RAM, 50 GB Storage" and "S3: 4 Core(s), 7 GB RAM, 50 GB Storage
" for "x2" and "x4" scaling respectively. 
For horizontal scaling, it uses the same **instance** for all "scaling", and instead increases the number of instances. I.e. **4 x "S1: 1 Core(s), 1.75 GB RAM, 50 GB Storage"**.


**I then compared the horizontal and vertical scaling on the "Premium v3" tier.**

The setup of **instances** and **number of instances** is the same as with the price comparison on the "Standard" tier, as explained above.
The prices for horizontal and vertical scaling are the same, but you do get more storage with horizontal scaling (this is also true for the "Standard" tier).

The instances used for vertical scaling are:
- x1 scaling = P1V3: 2 Core(s), 8 GB RAM, 250 GB Storage   <----- also used for horizontal scaling
- x2 scaling = P2V3: 4 Core(s), 16 GB RAM, 250 GB Storage
- x4 scaling = P3V3: 8 Core(s), 32 GB RAM, 250 GB Storage



**Finally I compared the prices for horizontal vs vertical scaling with VMs**

The setup of **instance** and **number of instances** is the same as with both price comparison for an **App Service**. I.e. the instance **D1 v3: 1 vCPUs, 3.5 GB RAM, 50 GB Temporary storage** is used for "x1 scaling scaling" and "x1, x2 and x4 horizontal scaling". Horizontal scaling changes the number of instances instead of the instance.

The instances used for vertical scaling are:
- x1 scaling = D1 v3: 1 vCPUs, 3.5 GB RAM, 50 GB Temporary storage   <----- also used for horizontal scaling
- x2 scaling = D2 v3: 2 vCPUs, 7 GB RAM, 100 GB Temporary storage
- x4 scaling = D3 v3: 4 vCPUs, 14 GB RAM, 200 GB Temporary storage



<!-- ### For an **App Service**?

### For a VM (Virtual Machine)?
 -->

## Which Azure App Service plans support app scaling?

**Azure App Service Autoscaling** can only scale in and out.

Not all App Service Plan **pricing tiers** support autoscaling.

For **Azure App Service** the **Free**, **Shared** and **Basic** tiers (which are meant for trialing, testing and development) do not support **Azure Auto Scale**, but the **Basic** tier supports up to 3 instances. The **Standard**, **Premium** and **Isolated** tiers all support **Auto Scale** and up to 10, 30 (in certain regions) and 100 instances respectively.

Funnily enough, even though the "Free" tier doesn't support more than **1 instance**, Microsoft's own **Pricing Calculator** allows you to select more than **1 instance** on the "Free" tier. Loop hole? 😁

![](/Molnapplikationer-Blogg/data/images/exercise-10-scaling/azure-pricing-weird-bug-free-tier-number-of-instances.png)


## Sources & Links
- [Pagely - Scaling up vs scaling out](https://pagely.com/blog/scaling-up-vs-scaling-out/)
- [Scaling out vs scaling up - Azure.Microsoft.com](https://azure.microsoft.com/sv-se/overview/scaling-out-vs-scaling-up/)
- [Azure App Service pricing - Azure.Microsoft.com](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)
- [Scale up an app in Azure App Service - Docs.Microsoft.com](https://docs.microsoft.com/en-us/azure/app-service/manage-scale-up)
- [Scale apps in Azure App Service - Docs.Microsoft.com](https://docs.microsoft.com/en-us/learn/modules/scale-apps-app-service/)
- [Scale up vs Scale out: What’s the difference? - opsani.com](https://opsani.com/blog/scale-up-vs-scale-out-whats-the-difference/)
- [Azure Pricing Calculator - Azure.Microsoft.com](https://azure.microsoft.com/en-us/pricing/calculator/)


[url-id]: url
