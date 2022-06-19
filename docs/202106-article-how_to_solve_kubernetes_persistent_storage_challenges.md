**Source: [TheNewStack.io](https://thenewstack.io/how-to-solve-kubernetes-persistent-storage-challenges/) - Date: 14/06/2021 - Author: Romuald Vandepoel**

Kubernetes and containerization can feel transformative compared to traditional cloud deployment approaches — or at least, that’s how it comes across in development environments. However, platform managers, heads of infrastructure and other professionals tasked with overseeing the production environment know that the transformative benefits of containerization are limited.

That’s because the fast failover, scalability, resource efficiency and other benefits of Kubernetes generally apply only for ephemeral workloads. Stateless applications — those that do not need to store data from one session to the next — do indeed perform admirably in a Kubernetes infrastructure. On the other hand, stateful applications that require persistent storage still depend on legacy infrastructure when in production. Thus, all those benefits associated with Kubernetes remain out of reach as long as your application relies on persistent workloads.

Fortunately, the challenges associated with Kubernetes’ persistent storage system can be solved with the right approach. Let’s explore why stateful applications are so difficult to work with in Kubernetes, how platform managers are responding and the alternatives they have at their disposal.

## The Kubernetes Persistent Storage Paradox
By design, containers work best with stateless applications. Kubernetes is able to create and remove containers in a rapid and dynamic manner because the applications within those containers come packaged with all of the dependencies they need to run. Regardless of where a new container is spun up — the same cluster or a different cloud provider — Kubernetes ensures that the application has access to the fundamental resources it needs to operate.

The dynamic creation and deletion of containers doesn’t work well for applications that need to persistently store data. As a stateful, containerized application is created or destroyed across a Kubernetes cluster, it must always know where its data is, have a high degree of access to that data and be able to ensure its integrity. This isn’t possible if an application’s stored state is destroyed every time its container is spun down.

Developers and platform managers want the best of both worlds: They want the fast failover, easy deployment and the efficiency of containers with the persistence of stateful workloads. There are approaches to establishing persistent storage for cloud-based applications. But, as we’ll see, a lot of the common approaches have their drawbacks.

## What Are Platform Managers Doing Instead?
### Building Their Own System
In industries where downtime and ephemeral-only solutions are not an option, such as the financial and healthcare industries, many invest a considerable amount of time and energy in deploying traditional storage systems to manage their stateful applications.

The chief downside to this approach is that it doesn’t provide the host of benefits that comes with using Kubernetes. It won’t be platform-agnostic, it needs to be run in a very specific environment, it will be difficult to scale, it’ll be more likely to experience downtime, and you’ll have to manage things like storage, failover and more in a highly manual fashion.

What’s more, no one can develop expertise in a legacy storage array until they join your organization. It takes time to train them on the system, and once that’s done, there are still only a handful of highly essential individuals at your organization who understand how the system works. It will also contribute to a major cloud anti-pattern, by reducing the self-service experience for the Kubernetes first-class citizen, the developer.

Perhaps most significantly, traditional storage systems lack the knowledge and innovation inherent in a massive open source project like Kubernetes. Your system will become depredated faster, problems will take longer to fix and updates will move at a slower pace.

### Building a Workaround
Obviously, not using Kubernetes comes with a lot of downsides. To provide Kubernetes with persistent storage and make it work with stateful applications, some developers choose to build arcane workarounds. But the problem with these workarounds is that they often counteract the benefits of using Kubernetes in the first place.

As an example, you might pin your application’s storage to its container, which means that Kubernetes can’t move your application if that node should fail. Or, you might identify a method to attach cloud storage into your containers, but this can slow your system down considerably and creates another potential point of failure.

## An Alternative Approach: Software-Defined, Cloud Native Storage
Ideally, your application could treat storage as another declared requirement. A containerized application doesn’t suddenly have trouble accessing, for instance, its declared load balancer when a node is spun down. Why can’t storage work similarly?

Fortunately, there is a class of solutions that provide such functionality. Storage orchestrators are software-defined, cloud native solutions that enable applications to treat storage as just another declared requirement. They provide storage that is attached to the same container as the application itself and is able to survive node failure.

These solutions work by aggregating all of a cluster’s stored data into a shared pool. When an application requests access to its data, storage orchestrators act as an intermediary. They fetch the relevant volume from the pool and provide it to whichever container within the cluster made the request. This way, when a given container is spun down or when a node goes offline, the stored data remains accessible in the cluster’s pool.

For platform managers, this means they don’t need to establish a complicated external storage solution for stateful workloads just to match their developers’ working environment. Instead, Kubernetes orchestrates their stateful applications in exactly the same way it does stateless applications.

## Identifying the Right Storage Orchestrator
Using a storage orchestrator will result in massive improvements in performance, reliability and scalability when compared to using a traditional storage array or a workaround solution. But that doesn’t mean each storage orchestrator is equally suitable for the same use cases.

We commissioned a study comparing the performance of four leading storage orchestrators:

* Longhorn
* OpenEBS
* Rook/CEPH
* Ondat

In this study, each orchestrator was subjected to the same battery of tests on the same hardware and software configuration to evaluate their performance under different circumstances. For platform managers looking to implement the optimal storage orchestrator to manage Kubernetes’ persistent storage in production, it can serve as a first step to identify the right solution. You can read the study results here.

