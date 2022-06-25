# Open Source Software at Ondat

## Curent Open Source Projects

### Discoblocks

| Logo | Status | Reference(s) | Website | Social(s) | Repo |
|------|--------|--------------|---------|-----------|------|
| ![](/images/discoblocks-logo.png){ width="100" } | Alpha  | :material-note-remove: | [discoblocks.io](https://discoblocks.io) | [:material-linkedin:](https://www.linkedin.com/company/discoblocks-io) | [:material-github:](https://github.com/ondat/discoblocks) | 

!!! summary
    Discoblocks is an open-source declarative disk configuration system for Kubernetes helping to automate CRUD (Create, Read, Update, Delete) operations for cloud disk device resources attached to Kubernetes cluster nodes.

??? info "Why?"

    Discoblocks can be leveraged by cloud-native data management platforms (like Ondat) to manage backend disks in the cloud.

    When using such a data management platform to overcome the block disk device limitation from hyperscalers, a new set of manual operational tasks needs to be considered like:  
    ```
    * provisioning block devices on the Kubernetes worker nodes  
    * partitioning, formatting, and mounting the block devices within a specific path (like /var/lib/vendor)  
    * capacity management and monitoring  
    * resizing and optimizing layouts related to capacity management  
    * decommissioning the devices in a secure way  
    ```

??? question "How?"
    At the current stage, Discoblocks is leveraging the available hyperscaler CSI (Container Storage Interface) within the Kubernetes cluster by introducing a CRD (Custom Resource Definition) per workload with   
    ```
    * capacity  
    * mount path within the Pod  
    * nodeSelector & podSelector  
    * upscale policy  
    * provision the relevant disk device using the CSI (like EBS on AWS) 
      when the workload deployment will happen  
    * monitor the volume(s)  
    * resize automatically the volume based on the upscale policy  
    ```

??? warning 
    An application could be using Discoblocks to get persistent storage but this option would not be safe for production as there will not be any data platform management to address high availability, replication, fencing, and encryption.

### Trousseau

| Logo | Status | Reference(s) | Website | Social(s) | Repo |
|------|--------|--------------|---------|-----------|------|
| ![](/images/trousseau-logo.png){ width="110" }  | Prod  | [:material-note-check:](https://finance.yahoo.com/news/trousseau-open-source-project-made-141300025.html)| [trousseau.io](https://trousseau.io) | [:material-linkedin:](https://www.linkedin.com/company/trousseau-io) [:material-twitter:](https://twitter.com/trousseauio) | [:material-github:](https://github.com/ondat/trousseau) | 

!!! summary
    Trousseau is an open-source project, based on [Kubernetes KMS provider design](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/). It is designed to be a framework for any KMS provider (see release notes).

??? info "Why?"

    Kubernetes platform users are all facing the very same question: how do we handle Secrets?

    While there are significant efforts to improve Kubernetes component layers, [the state of Secret Management isn't receiving much interest](https://archive.fosdem.org/2021/schedule/event/kubernetes_secret_management/). Using etcd to store API object definition & states, Kubernetes secrets are encoded in Base64 and shipped into the key value store database. Even if the filesystems on which etcd runs are encrypted, the secrets are still not.

    Instead of leveraging the native Kubernetes way to manage secrets, commercial and open source solutions solve this design flaw by leveraging different approaches all using different toolsets or practices. This leads to training and maintaining niche skills and tools increasing the cost and complexity of Kubernetes day 0, 1 and 2.

??? question "How?"
    By using the Kubernetes KMS provider framework to provide an envelope encryption scheme to encrypt secrets on the fly.  
    Once deployed, Trousseau will enable seamless secret management using the native Kubernetes API and kubectl CLI usage while leveraging an existing Key Management Service (KMS) provider.
