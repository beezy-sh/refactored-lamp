**Source: [TheNewStack.io](https://thenewstack.io/whats-the-biggest-gap-in-kubernetes-storage-architecture/) - Date: 16/07/2021 - Author: Romuald Vandepoel**

Year over year, Kubernetes consistently ranks among the most-loved platforms on Stack Overflow’s Developer Survey. It’s not hard to understand why. It offers easy scaling, great stability and reliability, works across many different environments. In short, it offers all the features that developers love.

Where it starts to lose its luster, however, is in production. The majority of applications out in the world are to some extent stateful, requiring persistent storage across one session to the next. Containers, and container orchestrators by extension, don’t tend to play well with persistent storage and stateful applications. Let’s explore why Kubernetes’ storage architecture is lacking and what users can do to host stateful applications in production.

The biggest gap in Kubernetes’ storage architecture? The fact that it doesn’t exist — there isn’t anything that could be appropriately called a storage architecture in Kubernetes.

Instead, Kubernetes provides a storage framework. The container orchestrator was never intended to provide storage in the first place.

There are a few reasons why this is the case and why it’s a good thing, even though it might make a production manager’s job more difficult.

First, containers are inherently ephemeral. The benefit of containerization lies in the ability for containers to be rapidly created or destroyed. Pinning a stateful workload to a container would prevent that from happening. In turn, this would prevent the fast failover, easy updates, lightweight provisioning and other benefits we derive from containers’ ephemeral nature.

This isn’t an insurmountable challenge, however. The major contributors in the Kubernetes ecosystem — specifically Google and Red Hat — could relatively easily release some definitive, “official” Kubernetes storage architecture.

This would incorporate Google’s and Red Hat’s particular design bias into the system, and it would contribute to vendor lock-in, something that would be against the open source nature of the Kubernetes project. If Google developed its own storage system for Kubernetes, that system would effectively become the default choice for storage even if it didn’t meet the needs of your particular application.

By providing a storage framework — the Container Storage Interface, or CSI — rather than a storage architecture, the Kubernetes project offers users freedom and flexibility at the cost of simplicity.

## What Does This Mean for Building Stateful Applications?
Because Kubernetes offers this framework for connecting a third-party storage architecture rather than its own storage architecture, Kubernetes users are faced with two choices:

* Use open source projects to build a bespoke storage solution for your application.
* Use a commercial-off-the-shelf storage solution that best fits your application’s needs.

The first approach, as we’ll see, comes with significant downsides.

### Why the Open Source Approach Is so Challenging
Let’s not kid ourselves, there are plenty of benefits to open source software. It’s free, it’s flexible, it leverages the wisdom of the crowd to overcome challenges and invent new solutions and more.

The open source model isn’t ideal for all applications and all industries. Although open source solutions are free initially, their cost comes in the form of higher risk, larger teams, slower development and similar hidden costs.

In the context of persistent storage for containerized applications, platform users would need to rely on a solution based on multiple open source projects to meet their needs. Each of these different projects is geared toward solving a specific problem, resulting in a patchy storage solution.

Furthermore, this stitching together all but guarantees something is going to break at some point, and when it does, the only experts you have to consult on how to fix the issue are the ones you’ve trained internally. Consider that the cost of downtime, excluding hard-to-quantify reputational damage, can exceed $5,600 per minute. Most organizations relying on an open source solution will want to train developers to specialize in the given application’s storage solution. Since these kinds of stitched-together solutions invariably require customization for each project, you’ll need to hire developers to support each project commensurately.

### Why Commercial Solutions Are Simpler
Actually, commercial solutions aren’t better than open source solutions — not inherently anyway. A commercial enterprise storage solution could still be a poor fit for your specific project, require internal expertise, require significant customization, break easily and come with all the drawbacks of an open source solution.

The difference is that where an open source solution is all but guaranteed to come with these headaches, a well-designed commercial enterprise storage solution won’t.

It isn’t a matter of commercial versus open source, rather it’s good architecture versus bad architecture. Open source solutions aren’t designed from the ground up, making it much more difficult to guarantee an architecture that performs well and ultimately saves money. Commercial storage solutions, however, are. This raises the odds that it will feature an architecture that meets enterprise requirements.

Ultimately, all this is to say that commercial storage solutions are a better fit for most Kubernetes users than open source ones, but that doesn’t mean you can skip the evaluation process. Whether open source or commercial, it’s important to determine whether any solution meets your performance requirements.

At Ondat, we commissioned a study comparing the performance of four leading persistent storage solutions. Download the study to see how they performed on a series of standardized tests.