---
layout: post
title:  "Exercise 1 The Cloud"
date:   2021-09-07 21:50:00 +0200
categories: cloud hosting
---
# Blog Post #1 - The Internet & the Cloud

## What is the cloud

In colloquial terms, the cloud is synonymous with Google Drive, Dropbox or Microsoft's Onedrive, and while these are indeed cloud products (Saas services?) the world of cloud hosting and development is a bit different. The cloud simply put is a collection of computers, or servers which are provided in various forms (e.g. Iaas, Paas or Saas). A company can rent these servers to run on their own premises (private cloud) or pay for services where the server is located and maintained in a datacenter (public cloud), or a combination of these two solutions (hybrid cloud). Companies can scale their apps according to their needs 

### Types
- Public
- Private
- Hybrid
### Service models
#### Iaas (Infrastructure as a service):
The cloud provider manages the hardware and the customers is requires to configure and maintain the operating system and network configuration themselves. The customer can select the hardware they require but rent it from the provider who is also responsible for maintaining and supporting the hardware.

#### Paas (Platform as a service):
The cloud provider manages the hardware, the Virtual Machines and the network configuration and the customer deploys their software in the provided environment (VM). Paas is more agil and comes with some benefits over Iaas:
- The customer is not required to configure the server(s) themselves.
- The customer can focus on developing the cloud app.
##### Paas disadvantages:
Platform limitations

#### Saas (Software as a service):
The cloud provider in this case is responsible for everything (hardware, VM, OS, network and software) except the data which is provided by the customer.

## What advantages and disadvantages can the cloud introduce?
### Benefits:
#### General benefits
- **High availability and redundancy:**
- High availability ensure a certain level of uptime, say 99.9%. This is accomplished by having redundant components which can take over in case of failure.
- **Scalability:** Cloud apps can be scaled both horizontally (number of instances/VMs) and vertically (number of virtual CPUs or Cores, amount of RAM and storage).
- **Elasticity:** Clouds apps can be automatically configured to scale as demand increases.
- **Agility:** Cloud apps can be configured and deployed quickly to meet changing requirements.
- **Geo-distribution:** Cloud apps can easily be deployed to datacenters across to the globe which provides better performance and
- **Disaster recovery:** Cloud-based backup solutions and geo-distribution and replication of data ensures redundancies in-case of disasters.
- **Resilience:** While the cloud doesn't inherently make the customers app/data more resilient (it entirely depends on what services the customers buys), it is easier and cheaper to create a redundancy plan which once in place could be managed by the cloud provider. A redundancy plan could include buying a backup service or distributing the app/data to multiple datacenters across the country or world.

#### Financial benefits:
Privately hosting a service or app necessitates purchasing, maintaining and supporting your own hardware and infrastructure, this means high up-front costs and continuous maintenance/support costs. Compare that to a public cloud solution where the provider is responsible for all costs of purchasing, maintaining and support the necessary hardware/infrastructure and the customer only pays for the resources they use.

### Disadvantages
The customer has to hand over their data and information to a 3rd party, which raises questions regarding security and privacy.



##  Price comparison of Cloud service providers
### The Exercise:
The exercise was to find a couple of cloud solution providers and compare their monthly price to that of the three "big giants"; Azure, AWS and Google Cloud.
The minimum requirements are; Linux, 2 CPUs, 8 GB RAM and 10 GB storage. The Virtual Server should be able to host a simple website and a simple database.
#### Notes:
_The exercise in full can be found [here][exercise]._

### Our conclusion:
We looked at three Swedish cloud providers in addition to the "big three"; AWS, Azure and Google Cloud. This image below shows all of them.
![Azure Virtual Server price estimate](/assets/images/cloud-providers-price-comparison-spreadsheet.png)
We came to the conclusion that the "big three" were cheapest options with one exception, Bahnhof. Whether or not this is true when all cloud providers are taken into account is uncertain since the sample size is far too small to make any final conclusions.
The "Big Three" do appear to have the most configurable services, and since these providers offer so many additional services under their cloud service umbrella you could conclude that either of the "big three" is a safe bet for any startup.

#### Below are the price estimates for the "big three" cloud providers:
##### Azure:
![Azure Virtual Server price estimate](/assets/images/azure-virtual-server-price-estimate.png)
##### AWS:
![Azure Virtual Server price estimate](/assets/images/aws-virtual-server-price-estimate.png)
##### Google Cloud:
![Azure Virtual Server price estimate](/assets/images/google-cloud-virtual-server-price-estimate.png)


## Sources
[Azure high availability][source-1]
[Advantages and Disadvantages of Cloud Computing][source-2]
[What is the Cloud? Soft and Fluffy Edition - Computer Stuff They Didn't Teach You #10][source-3]


[exercise]: https://pgbsnh20.github.io/PGBSNH20-molnapplikationer/cloud-lectures/introduction#pris-uppgift
[source-1]: https://cloud.netapp.com/blog/azure-high-availability-basic-concepts-and-a-checklist
[source-2]: https://www.stratospherenetworks.com/advantages-and-disadvantages-of-cloud.html
[source-3]: https://www.youtube.com/watch?v=BO6jvQ88ICQ
