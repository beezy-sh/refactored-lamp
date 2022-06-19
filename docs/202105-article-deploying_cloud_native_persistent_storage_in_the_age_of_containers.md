**Source: [TheNewStack.io](https://thenewstack.io/deploying-cloud-native-persistent-storage-in-the-age-of-containers/) - Date: 27/05/2021 - Author: Romuald Vandepoel**

The growth of software-defined, cloud native persistent storage is fueled by the many modern organizations that use containers for lightweight and portable app deployment — often needing a compatible storage strategy. By adopting container systems such as Kubernetes, users benefit from a range of capabilities from application scaling, failover, deployment and automation to service discovery, load balancing and self-healing. These became critical in an era where technology agility and efficiency are now top priorities for IT teams everywhere.

When deploying cloud native storage, organizations should be making decisions based on important factors that have the potential to make or break both the deployment process itself and the storage solution’s usefulness.

## Why Demand for Cloud Native Storage Accelerated

The impact of COVID-19 and the need to focus on rapid tech-led change across just about every industry has further accelerated the requirement for containers and cloud native storage. In the past year, data-dependent organizations have become even more focused on digital transformation and need to process and move vast volumes of data to the cloud more quickly than ever to fulfill their technology and business goals.

In doing so, they also want to store their data in their infrastructure of choice, irrespective of whether that’s in the cloud, on-premises or hybrid, and do so across a range of use cases, including high availability for applications and persistent volumes for Kubernetes. The result is a burgeoning demand for software-defined, cloud native storage. According to recent research from Technavio, it’s a market that will grow by $42.79 billion during 2020-2024, with 35% of this growth contributed by North America.

## Storage Deployment Decisions: Important Considerations
In particular, cloud native investment choices should take into account the prioritization of performance monitoring, data placement, technology upgrades and high availability. In doing so, it becomes possible to deliver the benefits of cloud native infrastructure across the board.

Looking at each of these five issues more closely can help smooth the planning and implementation process:

### Monitoring for Errors and Performance
Teams often focus primarily on application development, particularly at the “Day 2” stage of the software lifecycle (where the app is sufficiently advanced that it adds real benefits to the organization). A single-minded focus on application development sometimes prevents the organization from effectively monitoring for errors and performance problems — including any that involve storage. It’s not uncommon for legacy infrastructure to lack the instrumentation necessary to monitor the performance of storage systems as a result of this mindset

The knock-on effect of this is that without insight into performance issues, important details are missed and IT teams can’t diagnose problems — let alone quickly resolve them. On the other hand, cloud native storage systems should place users back in control by supplying them with performance metrics and data so they can identify and act on any issues that this information reveals.

### Data Placement
The challenges associated with application deployment can also be addressed by implementing a Kubernetes storage system optimized for effective data placement. In particular, data should be placed as close to the application using that data as possible in order to maximize performance.

In addition, cloud native storage should also replicate data across availability zones so that users are immediately provided with alternative access in the event of a zone going offline, boosting overall reliability and uptime.

### Technology Upgrades
Application upgrades are integral to the infrastructure lifecycle and may include a variety of changes such as new versions of Kubernetes, data and applications. Even though Kubernetes is a highly effective orchestration tool and includes upgrade management, the storage system also needs to be flexible enough to work with Kubernetes during any ongoing upgrade process.

It’s useful to seek out storage systems that can offer what is called a “Kubernetes Operator,” which is a tool that automates deployment and product lifecycle for the system for a more efficient approach.

### High Availability
For most organizations, high availability is a must-have from the outset in order to deliver the redundancy and data protection capabilities required in the event of infrastructure downtime or network problems.

By far, the best approach is to plan for the likelihood that problems will occur — and in doing so, application data should be set up to automatically failover to other nodes to minimize or eliminate disruption. Cloud native storage can facilitate this.

### From Monitoring to Observability
Last but not least, ensuring IT teams have the tools and technologies to proactively monitor and log storage infrastructure is the only way to make the challenges associated with Kubernetes storage easy to address. This is critical because, in today’s stateful applications, storage problems are among the key reasons behind application performance problems and availability.

Observability will provide “Day 2” with a unique position to correlate metrics and logs gathered from the entire stack identifying broken patterns, bottlenecks or unexpected outliers from the interaction of all the components during normal operations and failures leveraging historical data to compare behavior changes between releases.

Addressing these from the outset can have a transformative impact.