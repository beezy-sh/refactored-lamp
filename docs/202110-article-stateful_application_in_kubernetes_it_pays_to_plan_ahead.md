**Source: [TheNewStack.io](https://thenewstack.io/why-plan-stateful-application-storage/) - Date: 27/10/2021 - Author: Romuald Vandepoel**

Abraham Lincoln once reputedly said, “Give me six hours to chop down a tree, and I will spend the first four sharpening the axe.” Lincoln knew not to neglect the time and effort to perform due diligence before starting any project, whether that’s chopping down a tree or developing a stateful application.

Any development project is going to feature its unexpected twists and turns — scope expands, requirements change, unforeseen dependencies are uncovered and more. But the key to managing these unexpected challenges is to plan for those elements you can forecast and control.

When developing stateful applications, those elements are your Kubernetes implementation, modules, libraries, plugins and solutions being used to support applications in production. Let’s dive into how we can all benefit from fully planning out the stateful application’s journey.

## What Does Playing It by Ear Look Like?
Let’s play out the typical process for enterprises deploying a Kubernetes-based stateful application.

You’ll select your Kubernetes implementation first. Maybe you want to go with a pure cloud solution, like Google Kubernetes Engine (GKE), Amazon Elastic Kubernetes Service (EKS) or Azure Kubernetes Service (AKS). Or perhaps you want to use your on-premises data center for solutions like RedHat’s OpenShift or Rancher.

You’ll need to evaluate all the different components required to get your cluster up and running. For instance, you’ll likely have a preferred container network interface (CNI) plugin that meets your project’s requirements and drives your cluster’s networking.

Once your clusters are operational and you’ve completed the development phase, you’ll begin testing your application. But now, your platform team is struggling to maintain your stateful application’s availability and reliability.

As part of your stateful application, you’ve been using a database like Cassandra, MongoDB or MySQL. Every time a container is restarted, you begin to see errors in your database. You can prevent these errors with some manual intervention, but then you’re missing out on the native automation capabilities of Kubernetes.

### The Consequences of Thinking About Storage too Late
Just like your network plugin, you need a storage plugin to enable a stateful application like your hypothetical database to function properly in Kubernetes.

You may have needed a network plugin just to get your cluster up and running in the first place, so naturally, you implemented this part of your project first. The need for a storage plugin only emerged later once you started to test your application in its acceptance environment.

So, you begin investigating storage solutions, almost in a rush to find a workable solution. Maybe you try a few out, only to discover they have dependencies that won’t work with your organization. To get to market faster, many storage solutions depend upon kernel modules as development shortcuts, but now you, the end user, are paying the price. Your choices are to cobble together a workaround, rely on a subpar solution that happens to meet your particular implementation or maybe even develop your own solution based on open source technologies.

If your application is hosted in a hybrid environment, you’ll also be limited by whatever capabilities your on-premises tech stack permits. Depending on your organization’s legacy infrastructure, you may not be able to benefit from all the capabilities you’d expect in a Kubernetes data plane.

Ultimately, the result is a project that was on schedule right up until the end. Worse than missed timelines, such projects may now need a dedicated support team to manage the storage infrastructure. Once deployed, the project may not always be available, may experience high latency, see excessive data errors or underperform in another way.

The bottom line? Failing to fully plan for the storage needs of a stateful application means less revenue due to unreliable performance and greater costs due to maintenance and firefighting.

## What About Planning It out from the Start?
No developer wants to make ad hoc design decisions, and the same goes for infrastructure teams. There are factors that can make it easy for a development project to become more improvisational than is merited, especially when working with stateful applications on Kubernetes.

Chiefly, this is because the preproduction and production environments are typically the responsibility of another team entirely. Everything might be planned out in the development environment, but the novel requirements of production can throw a wrench in the gears. Since a stateful application storage solution is one such requirement, it’s important for developers to collaborate with their platform managers as early as possible, even though storage isn’t an issue during the development process.

When platform managers are consulted from the start, developers can plan which solutions their application will rely on for its full life cycle. Just as developers pick their preferred network plugin at the start of their project, they can identify a storage plugin that meets their needs.

This way, developers won’t discover that they don’t have the right dependencies or environment configuration in place. They can select a storage solution whose dependencies match the project’s requirements, or better yet, they can pick a platform-agnostic solution like Ondat.

What’s more, once developers begin to investigate the incorporation of a storage solution to support their stateful applications, they’ll likely find that planning for such a solution is easier than they thought. To support the unique requirements of a container-based deployment, many applications and components are available as Kubernetes operators. These operators typically include integration hooks for storage solutions.

## Evaluate Solutions for Your Stateful Apps Early
Knowing that it’s necessary to plan for your application’s storage requirements to avoid any last-minute delays is just the start. Once you begin to investigate different storage solutions, it’s important to be able to evaluate them.

Since storage often falls outside the developer’s purview, we’ve commissioned a study to help developers evaluate four of the top Kubernetes-based cloud native storage solutions. If you’ve just started to plan for your stateful application development project, we highly recommend checking it out.