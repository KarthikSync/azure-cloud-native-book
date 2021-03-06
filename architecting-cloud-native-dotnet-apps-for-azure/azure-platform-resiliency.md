---
title: Cloud Infrastructure Resiliency with Azure
description: Architecting Cloud Native .NET Apps for Azure | Cloud Infrastructure Resiliency with Azure
ms.date: 06/30/2019
---

# Azure Platform Resiliency
=========================

Building a reliable application in the cloud is different from traditional on-premises application development. While historically you may have purchased higher-end hardware to scale up, in a cloud environment you scale out instead of up. Instead of trying to prevent failures, the goal is to minimize  their effects and keep the system stable.

That said, reliable cloud applications exhibit distinct characteristics:

-   They are resilient�and recover gracefully from problems and continue to function

-   They are highly available (HA)�and run as designed in a healthy state with no significant downtime

Understanding how these characteristics work together � and how they affect cost � is essential to building a reliable cloud native application. We�ll next look at ways that you can build resiliency and  availability into your cloud native applications leveraging features from the Azure cloud.

### Design with Redundancy

Failures vary in scope of impact. A hardware failure, such as a failed disk, can affect a single node in a cluster. A failed network switch could affect an entire server rack. Less common failures, such as loss of power, could disrupt a whole datacenter. Rarely, an entire region becomes unavailable.

Redundancy is one way to provide application resilience. The exact level of redundancy needed depends on your business requirements and will affect both cost and complexity of your system. For example, a multi-region deployment is more expensive and more complex to manage than a single-region deployment. You
will need operational procedures to manage failover and failback. The additional cost and complexity might be justified for some business scenarios and not others.

To architect redundancy, you need to identify the critical paths in your application. And, then determine if there is redundancy at each point in the path? If a subsystem should fail, will the application fail over to something else? Finally, you need a clear understanding of those features built into the Azure cloud platform that you can leverage to meet your redundancy requirements. Here are recommendations for architecting redundancy:

-   *Deploy multiple instances of services.*�If your application depends on a single instance of a service, it creates a single point of failure. Provisioning multiple instances improves both resiliency and scalability. When hosting in Azure Kubernetes Service, you can declaratively configure redundant instances (replica sets) in the Kubernetes manifest file. The replica count value can be managed programmatically, in the portal or through autoscaling features, which we discuss later on.

-   *Leveraging a load balancer.*�Load-balancing distributes your application's requests to healthy service instances and automatically removes unhealthy instances from rotation. When deploying to Kubernetes, load balancing can be specified in the Kubernetes manifest file in the Services section.

-   *Plan for multiregion deployment.* �If your application is deployed to a single region, and the region becomes unavailable, your application will also be unavailable. This may be unacceptable under the terms of your application's service level agreements. Instead, consider deploying your application and its services across multiple regions. For example, an Azure Kubernetes Service (AKS) cluster is deployed to a single region. To protect your system from a regional failure, you might deploy your application to     multiple AKS clusters across different regions and use the paired regions feature to coordinate platform updates and prioritize recovery efforts.

-   *Enable geo-replication.* Geo-replication for services such as�Azure SQL Database�and�Cosmos DB�will create secondary replicas of your data across multiple regions. While both services automatically replicate data within the same region, geo-replication protects you against a regional outage by   enabling you to fail over to the secondary region.

    Another best practice for geo-replication centers around storing container images. To deploy a service in AKS, you need to store and pull the image from a repository. Azure Container Registry integrates with AKS and can securely store container images. To improve performance and availability, consider geo-replicating your images to a registry in each region where you have an AKS cluster. Each AKS cluster then pulls container images from the local container registry in the same region as shown in Figure 6-6:

    ![Replicated Resources Across Regions](media/replicated-resources.png)

**Figure6-6**. Replicated Resources Across Regions

�

-   *Implement a DNS traffic load balancer.*�Azure Traffic Manager�provides high-availability for critical applications by load-balancing at the DNS level. It can route traffic to different regions based on geography, cluster response time and even the health of your application's endpoints. For     example, Azure Traffic Manager can direct customers to the closest AKS cluster and application instance. If you have multiple AKS clusters in different regions, use Traffic Manager to control how traffic flows to the     applications that run in each cluster. Figure 6-7 depicts this scenario.

    ![AKS and Azure Traffic Manager](media/aks-traffic-manager.png)

**Figure 6-7**. AKS and Azure Traffic Manager

### Design for Scalability

The cloud thrives on scaling. The ability to increase/decrease system resources to address increasing/decreasing system load is a key tenet of the Azure cloud. But, to effectively scale an application, you need an understanding of the scaling features of each Azure service that you include in your application.  Here are recommendations for effectively implementing scaling in your system.

-   *Design for scaling.*�An application must be designed for scaling. To start, services should be stateless so that requests can be routed to any instance. Having stateless services also means that adding or removing an instance does not adversely impact current users.

-   *Partition workloads*.�Decomposing domains into independent, self-contained microservices enable each service to scale independently of others. Typically, services will have different scalability needs and requirements. Partitioning enables you to scale only what needs to be scaled without the unnecessary cost of scaling an entire application.

-   *Favor scale-out.* Cloud-based applications favor scaling out resources as opposed to scaling up. Scaling out, often called horizontal scaling, involves adding more service resources to an existing system in order to meet and share a desired level of performance. On the other hand, scaling up, often called vertical scaling, involves replacing existing resources with more powerful hardware (more disk, memory and processing cores). Scaling out can be invoked automatically with the auto-scaling features available in some Azure cloud resources. Scaling out across multiple resources also adds redundancy to the overall system. Finally scaling up a single resource is typically more expensive than scaling out across many     smaller resources. Figure 6-8 below shows the two approaches.

![Scale Up vs. Scale Out](media/scale-up-scale-out.png)

**Figure 6-8.** Scale Up vs. Scale Out

-   *Scale proportionally.*�When scaling a service, think in terms of *resource sets*. If you were to dramatically scale out a specific service, what impact would that have on backend data stores, caches and dependent services? Some resources such as Cosmos DB can scale out proportionally, while many other    cannot. You want to ensure that you do not scale out a resource that will to a point where it will choke other associated resources.

-   *Avoid affinity.* A best practice is to ensure a node doesn't require local affinity, often referred to as a *sticky session*. A request should be able to route to any instance. If you do need to persist state, it should be saved to a distributed cache, such as Azure Redis cache.

-   *Take advantage of platform autoscaling features.*�Use built-in autoscaling features whenever possible, rather than custom or third-party mechanisms. Use scheduled scaling rules, where possible, to ensure that resources are available without a startup delay, but add reactive autoscaling to the rules, where appropriate, to cope with unexpected changes in demand. For more information, see�Autoscaling guidance.

-   *Scale-up aggressively.* A final practice would be to scale up aggressively so that you can quickly meet immediate spikes in traffic without losing business. And, then scale down (remove unneeded resources) conservatory in order to keep the system stable. A simple way to implement this might be to set the cool down period (the time to wait between scaling operations) for adding resources to 5 minutes and the period for removing instances up to 30 minutes.

### Built-in Retry in services

We encouraged the best practice of implementing programmatic retry operations in an earlier section. Keep in mind that many Azure services and their corresponding client SDKs also include retry mechanisms. Here, we summarize retry features in the many of the Azure services we have discussed in this book.

-   *Azure Cosmos DB.* The DocumentClient class from the client API automatically retires failed attempts. The number of retries and maximum wait time are configurable. Exception thrown by the client API are either requests that exceed the retry policy or non-transient errors.

-   *Azure Redis Cache.* The Redis StackExchange client uses a connection manager class that includes retries on failed attempts. The number of retries, specific retry policy and wait time are all configurable.

-   *Azure Service Bus.* The Service Bus client exposes a RetryPolicy property that can be configured with a back-off interval, retry count and TerminationTimeBuffer which specifies the maximum time an operation can take. The default policy is 9 maximum retry attempts with a 30 second backoff period between attempts.

-   *Azure SQL Database.* Retry support is provided when using the Entity Framework Core library.

-   *Azure Storage.* The storage client library support retry operations. The strategies vary across Azure storage tables, blobs and queues. As well, alternate retries switch between primary and secondary storage services locations when read access geo-redundant storages is enabled.

-   *Azure Event Hubs.* The Event Hub client library features a RetryPolicy property which includes a configurable exponential backoff feature.


>[!div class="step-by-step"]
>[Previous](application-resiliency-patterns.md)
>[Next](resilient-communications.md)
