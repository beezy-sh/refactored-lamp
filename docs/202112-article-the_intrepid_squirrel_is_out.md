**Source: [Ondat.io](https://www.ondat.io/blog/the-intrepid-squirrel-is-out) - Date: 8/12/2021 - Author: Romuald Vandepoel**

![cat&squirrel](/images/the_intrepid_squirrel_is_out.png)

The Marketing and Product teams gave me a unique opportunity to write a blog post about the General Availability of Ondat 2.5. This is quite an honor and I will use it to raise my case regarding the usage of cool code names!

Today, we are proud to announce the General Availability of 'Intrepid Squirrel' running on the 2.5 branch of our Ondat core product.

Our Intrepid Squirrel is the result of our incredible Engineering teams, your work has been crucial in achieving this important milestone! You rock!

Releasing at this time of the year is perfect to provide features and improvements that help with the holiday season operational freeze. Our little adventurous friend gathered the best treats to relieve our customers not only from the seasonal challenges but from the general operational burden of running cloud and on-prem Kubernetes clusters.

What's in the tree dens? Our little friend's home is overflowing with good stuff! Here are my three favorite ones:

## "I am Tank. I'll be your operator"

It always pays to plan ahead and our Intrepid Squirrel knows it! This is the reason why our Operator has been entirely rewritten to pave the road to the future. Kubernetes is a fast-moving project with continuous improvements at every layer including the persistent storage one. The new operator provides our customer with a frictionless deployment and maintainability strategy, an agile adoption of new Kubernetes primitives while avoiding the deprecation traps, and an open door to new Ondat products!

## Happy Kubecuddle! 

Exploring and maintaining a new solution is not always as simple as it looks despite well-written documentation. That's why our adventurous squirrel decided to streamline the installation and operational processes by introducing a kubectl plugin to deploy and operate one or many Ondat data mesh clusters. Let's have a couple of examples:

Always good to have some preflight checks before having the "hum!" moment:

kubectl storageos preflight

![screenshot](/images/the_intrepid_screenshot.png)

What about deploying a self-evaluation or test environment with one easy command:

kubectl storageos install --include-etcd --admin-username rom --admin-password mysecretpassword
namespace/storageos-etcd created

```
kubectl get pod -n storageos-etcd
NAME READY STATUS RESTARTS AGE
storageos-etcd-0-qcq8h 1/1 Running 0 2m3s
storageos-etcd-1-d7lbv 1/1 Running 0 2m3s
storageos-etcd-2-p72tm 1/1 Running 0 2m3sstorageos-etcd-controller-manager-7c6df47dfb-5xlq5 1/1 Running 0 2m15s
storageos-etcd-proxy-64cf4f6556-dnfrg 1/1 Running 0 2m15s

kubectl get pod -n storageos
NAME READY STATUS RESTARTS AGE
storageos-api-manager-65f5c9dbdf-p4c92 1/1 Running 1 (52s ago) 61s
storageos-api-manager-65f5c9dbdf-wdg59 1/1 Running 0 61s
storageos-csi-helper-65dc8ff9d8-hmz4q 3/3 Running 0 61s
storageos-node-6z4zd 3/3 Running 0 103s
storageos-node-bvmrq 3/3 Running 0 103s
storageos-node-czbp9 3/3 Running 0 103s
storageos-node-fsqms 3/3 Running 0 103s
storageos-node-tmrm8 3/3 Running 0 103s
storageos-node-vbmml 3/3 Running 0 103s
storageos-operator-b7f6ff69-qrkwv 2/2 Running 0 2m17sstorageos-scheduler-6f86686756-wpvzf 1/1 Running 0 109s
``` 
Done! And there are more to look at... but your turn now!

## Replicas... replicas everywhere!

Whilst the cloud can abstract and simplify many infrastructure challenges, running a Kubernetes data plane in only one availability zone will result in a major disruption in event of a human or provider failure. Well in this release, our Intrepid Squirrel decided to extend his data home range with more than one availability zone, by detecting them with the native Kubernetes topology key or using our own custom key.  Allowing the system to then provision the primary volumes and replicas evenly across these availability zones.

In the event of a zone failure, our Rapid Recovery will elect one of the replicas as primary volume, call on the Kubernetes scheduler to restart the StatefulSet to the new zone, and start a new replica to honor the required replica count.

Also with some clever optimizations, our Intrepid Squirrel got faster, way faster, up to 45 times faster when provisioning new replicas and rejoining a volume group by leveraging concurrent data replications across multiple zones and optimizing the network resources.

Let's verify the available regions, create a PVC, scale the number of replicas to 2 (1 primary volume + 2 replicas = 3 zones), then check where are the volume and its replicas placed, hopefully in different availability zones:

```
kubectl get node -ojson |jq '.items[].metadata.labels'|grep "topology.kubernetes.io/zone"
"topology.kubernetes.io/zone": "us-central1-f"
"topology.kubernetes.io/zone": "us-central1-f"
"topology.kubernetes.io/zone": "us-central1-c"
"topology.kubernetes.io/zone": "us-central1-c"
"topology.kubernetes.io/zone": "us-central1-a"
"topology.kubernetes.io/zone": "us-central1-a"

cat pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: pvc-tap
labels:
storageos.com/topology-aware: 'true'
storageos.com/failure-mode: 'soft'
spec:
storageClassName: storageos
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 5Gi

kubectl apply -f pvc.yaml
persistentvolumeclaim/pvc-tap created

kubectl get pvc pvc-tap -owide
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE VOLUMEMODE
pvc-tap Bound pvc-03b55134-95b1-4609-a07b-3f73f115d0a3 5Gi RWO storageos 6m18s Filesystem

kubectl describe pvc pvc-tap
Name: pvc-tap
Namespace: default
StorageClass: storageos
Status: Bound
Volume: pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
Labels: storageos.com/failure-mode=soft
storageos.com/topology-aware=true
Annotations: pv.kubernetes.io/bind-completed: yes
pv.kubernetes.io/bound-by-controller: yes
storageos.com/storageclass: 27550a36-cf0f-489e-8169-bab05ceb1cc7
volume.beta.kubernetes.io/storage-provisioner: csi.storageos.com
Finalizers: [kubernetes.io/pvc-protection]
Capacity: 5Gi
Access Modes: RWO
VolumeMode: Filesystem
Used By: <none>
Events:
Type Reason Age From Message
---- ------ ---- ---- -------
Normal Provisioning 19s csi.storageos.com_storageos-csi-helper-65dc8ff9d8-hmz4q_e65aa0d3-0541-4901-9e07-5898677577b1 External provisioner is provisioning volume for claim "default/pvc-tap"
Normal ExternalProvisioning 19s persistentvolume-controller waiting for a volume to be created, either by external provisioner "csi.storageos.com" or manually created by system administrator
Normal ProvisioningSucceeded 18s csi.storageos.com_storageos-csi-helper-65dc8ff9d8-hmz4q_e65aa0d3-0541-4901-9e07-5898677577b1 Successfully provisioned volume pvc-03b55134-95b1-4609-a07b-3f73f115d0a3

kubectl describe pv pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
Name: pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
Labels: <none>
Annotations: pv.kubernetes.io/provisioned-by: csi.storageos.com
Finalizers: [kubernetes.io/pv-protection]
StorageClass: storageos
Status: Bound
Claim: default/pvc-tap
Reclaim Policy: Delete
Access Modes: RWO
VolumeMode: Filesystem
Capacity: 5Gi
Node Affinity: <none>
Message:
Source:
Type: CSI (a Container Storage Interface (CSI) volume source)
Driver: csi.storageos.com
FSType: ext4
VolumeHandle: 3ab351c2-0f75-4bab-8851-cf7c414811d5/4d65dddd-8109-4255-9639-edde066ab50d
ReadOnly: false
VolumeAttributes: csi.storage.k8s.io/pv/name=pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
csi.storage.k8s.io/pvc/name=pvc-tap
csi.storage.k8s.io/pvc/namespace=default
storage.kubernetes.io/csiProvisionerIdentity=1636678069021-8081-csi.storageos.com
storageos.com/failure-mode=soft
storageos.com/nocompress=true
storageos.com/topology-aware=true
Events: <none>

kubectl label pvc pvc-tap storageos.com/replicas=2
persistentvolumeclaim/pvc-tap labeled
romuald_vandepoel@cloudshell:~/tap (shaped-complex-318513)$ kubectl describe pvc pvc-tapName: pvc-tap
Namespace: default
StorageClass: storageos
Status: Bound
Volume: pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
Labels: storageos.com/failure-mode=soft
storageos.com/replicas=2
storageos.com/topology-aware=true
Annotations: pv.kubernetes.io/bind-completed: yes
pv.kubernetes.io/bound-by-controller: yes
storageos.com/storageclass: 27550a36-cf0f-489e-8169-bab05ceb1cc7
volume.beta.kubernetes.io/storage-provisioner: csi.storageos.com
Finalizers: [kubernetes.io/pvc-protection]
Capacity: 5Gi
Access Modes: RWO
VolumeMode: Filesystem
Used By: <none>

storageos describe volumes pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
ID 4d65dddd-8109-4255-9639-edde066ab50d
Name pvc-03b55134-95b1-4609-a07b-3f73f115d0a3
Description
AttachedOn
Attachment Type detached
NFS
Service Endpoint
Exports:
Namespace default (3ab351c2-0f75-4bab-8851-cf7c414811d5)
Labels csi.storage.k8s.io/pv/name=pvc-03b55134-95b1-4609-a07b-3f73f115d0a3,
csi.storage.k8s.io/pvc/name=pvc-tap,
csi.storage.k8s.io/pvc/namespace=default,
storageos.com/failure-mode=soft,
storageos.com/nocompress=true,
storageos.com/replicas=2,
storageos.com/topology-aware=true
Filesystem ext4
Size 5.0 GiB (5368709120 bytes)
Version Nw
Created at 2021-11-12T01:07:08Z (16 minutes ago)
Updated at 2021-11-12T01:12:09Z (11 minutes ago)

Master:
ID d627cf2e-e4d6-4b2b-a595-45f3334762cf
Node gke-cluster-1-default-pool-c6f6c5c1-ctcj (08d37e34-81c0-45a7-8507-355deeac9346)
Health online

Replicas:
ID cd47db1f-ff00-4dfd-afd9-5ee06e1520ec
Node gke-cluster-1-default-pool-9aba0152-hg08 (00c68f1c-2289-468e-a394-c60049936f58)
Health ready
Promotable true

ID e9d61756-a0a2-403e-a38e-cdccc7ddb02c
Node gke-cluster-1-default-pool-d81d0b14-9vfn (d7500964-ae30-451e-bcf2-93f8bd77a3bf)
Health ready
Promotable true

kubectl get node {gke-cluster-1-default-pool-c6f6c5c1-ctcj,gke-cluster-1-default-pool-9aba0152-hg08,gke-cluster-1-default-pool-d81d0b14-9vfn} -ojson |jq '.items[].metadata.labels'|grep "topology.kubernetes.io/zone"
"topology.kubernetes.io/zone": "us-central1-c"
"topology.kubernetes.io/zone": "us-central1-f"
"topology.kubernetes.io/zone": "us-central1-a"
```

Done! And now, it would be great to "kill" a zone to perform disaster and resilience testing... but your turn!

## More... More! We want more!

Well, our Intrepid Squirrel is not out of treats! There are more features and improvements to investigate.

Visit our Docs website to get a deeper look at our [Release Notes](https://docs.ondat.io/v2.5/docs/reference/release_notes/) and let's hope the marketing department takes the hint and starts to use cool code names for releases!!!