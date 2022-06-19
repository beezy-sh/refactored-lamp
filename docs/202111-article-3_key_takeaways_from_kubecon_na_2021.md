**Source: [Ondat.io](https://www.ondat.io/blog/top-3-key-takeaway-from-kubecon-na-2021) - Date: 2/11/2021 - Author: Romuald Vandepoel**

When was my last in-person event? Oh! Yep, I remember, it was Fosdem 2020 in Brussels back from a trip to the US just before the chaos...

Well, after almost 20 months of virtual events, having such a major event in-person is quite a mental shift, shaking our remote habits of being overwhelmed content viewers.

KubeCon North America 2021 didn't disappoint from a venue and organization perspective. It gave us a shiny blue sky full of colors with an incredible amount of sessions, breakouts, and co-located events. Thank you CNCF and all the amazing staff ensuring our safety.

Obviously, I could not come back from such an amazing experience without sharing with you my top 3 takeaways. Buckle up and let's go!

## Look... Look! Did you see it?

Monolithic applications are quite predictable by design. This statement does not really fit with true distributed microservice architecture. As a matter of fact, if you are not already thinking about Chaos Engineering, you might need to revise your strategy to guarantee a smooth ride in the Kubernetes world.

![kubeconna01](/images/kubecon_na_2022_01.jpg)

The practice is often wrongly associated with monitoring and reduced to troubleshooting a defect at any layer of the stack. Leveraging deep component monitoring with correlations of other data sources like logs, observability enables application and platform teams to find out why something went wrong or a performance decrease happened.

Having such understanding of identifying and measuring benefits or downsides of bug fixing, scaling up and down resources, introducing caching mechanisms, changing infrastructure components, along another zillion of changes allows to proactively avoid massive failure in production.

These insights are valuable for Product Managers to provide to end-customers, application, infrastructure, and engineering teams with clear outcomes on the business, operations, and development efforts in short close feedback loops.

No wonder why every single person that I met at KubeCon discussed observability. Sessions and vendors like Honeycomb, Komodor, and StreamNative were definitely present to support their needs with value-add solutions.

## More love for your workload

The first-class citizens of Kubernetes are the applications! To support that statement, there is a need to give more love to the overall workload lifecycle and it can take a different approach but it should be simple as A, B, C.

A. Application definitions are multiplying based on the specifics of the infrastructure, CI/CD, or GitOps choices not even mentioning everything related to monitoring, audit, and compliance requirements.

Would it not be fantastic to have a universal application definition that would be agnostic of all the above? I present you AppOps with Shipa! No matter if you're using Pulumi, Terraform, Argo, or Flux, Shipa provides you with a common framework definition for your Applications addressing any of your preferred tooling and moving from one to another based on your organization needs and platform choices. Sweet!

B. Some of my projects had to deal with thousands of applications having up to 10 different versions in the registry. This can quickly become an unforeseen cost both from registry and worker node storage perspective.

Well, the container image fitness coach is in town at a new place called Slim.ai! If you dream of reducing your corporate container images from 1.5GB to 140MB while still preserving an enterprise lifecycle, adding container internals auditing, and optimizing and minifying the footprint, this is the place to go!

C. How to receive love from your CISO? Introducing new ways of working, deploying, and managing the lifecycle of business-critical applications is not easy for everyone and especially for CISO. In fact, while it took a great amount of time to build governance and compliance within the "old" world, the "new" one is shaking the tree quite hard in terms of... actually everything due to a massive shift left.

![kubeconna01](/images/kubecon_na_2022_02.jpg)

How can you possibly leverage this shift to make it a win with CISO? Most likely by integrating within your CI/CD pipelines all the lifecycle, security, and vulnerability auditing, while benchmarking your builds against SOC 2 and CIS and you can do all of these with Fairwinds.

## My precious!

During the week, I had more than 50 customers sharing that they are postponing their deployment of stateful application in production due to bad experiences with Kubernetes data management layers... or maybe the lack of a native one.

Wearing my capt'n obvious uniform, I will make a super bold statement... or not!

While the end goal is to run stateless applications on Kubernetes, somewhere within the organization, there are a good amount of stateful applications storing and feeding data back in any shape or form. Also, transformation journeys start in most of the cases with a lift and shift providing more flexibility with costs and efforts to refactor applications.

Considering the above, old enterprise vendors will recommend to not run stateful applications or at least some of them, like databases, either because their Kubernetes data layer is not performant enough (like an object store based one) or in benefit of their managed database services (like RDS or any equivalent). The Kube native vendors have engineered a hyper performant cloud software-defined data management platform supporting both self-hosted large MongoDB along with legacy applications or managed database services with real value-add like with Timescale.

Adam and I were there to debunk the urban legends and explained how Ondat can solve your persistent storage challenges with legacy and cloud native applications.

