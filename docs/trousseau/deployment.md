

## Overview 
Trousseau installation deploys and maintains its component as container using native Kubernetes API and mechanism along with the standard KMS (Key Management Service) tooling.   

In production-grade environment, a *separation of duties* will most likely be in place dividing the work between a DevOps team in charge of the Kubernetes side while a Security team will be in charge of the KMS one. The installation guide will mention who is doing what with ***Platform team*** and ***Security team*** mark. 

The following workflow shows the installation steps and owners:

| # | Activity |Responsible Team|
|---|----------|----------------|
| 1 | [Create a Kubernetes ServicesAccount](#create-a-kubernetes-serviceaccount-kubernetes-admins) | ***Platform Team***|
| 2 | [HashiCorp Vault Kubernetes Authentication & Transit engine](#hashicorp-vault-kubernetes-authentication--transit-engine-kubernetes-admins--security-admins) | ***Platform Team*** <br> ***Security Team***|
| 3 | [Create a Trousseau Kubernetes ConfigMap](#create-a-trousseau-kubernetes-configmap-kubernetes-admins) | ***Platform Team***|
| 4 | [Deploy the Trousseau DaemonSet](#deploy-trousseau-daemonset-kubernetes-admins) | ***Platform Team***|
| 5 | [Enable Trousseau KMS provider in ```kube-apiserver```](#enable-trousseau-kms-provider-in-kube-apiserver-kubernetes-admins)| ***Platform Team***|
| 6 | [Create and verify Secret encryption with Trousseau and HashiCorp Vault](#create-and-verify-secret-encryption-with-trousseau-and-hashicorp-vault-kubernetes-admins) | ***Platform Team*** | 
| 7 | [Replace existing Kubernetes secrets with encrypted ones using Trousseau](#replace-existing-kubernetes-secrets-with-encrypted-ones-using-trousseau-kubernetes-admins) | ***Platform Team***|


## Requirements
The following are required:  

- a HashiCorp Vault instance (Community or Enterprise)
- a HashiCorp Vault token
- a SSH access to the control plane nodes as with root permissions
- an internet access or follow the disconnected requirements
- ```kubectl``` CLI on an bastion host with access to Kubernetes
- ```vault``` CLI an bastion host with access to the HashiCorp Vault instance

### Test/Dev environment
- a working Kubernetes cluster with 1 control plane node & 1 worker node
- a dev/test HashiCorp Vault Community
- deployed within a Kubernetes (different as Trousseau to avoid a death dependency circle) 
- outside of Kubernetes as a virtual machine or a [cloud instances](https://cloud.hashicorp.com/)

### Production environment
- a working Kubernetes cluster with 3 control plane nodes & 1 worker node
- a production-grade HashiCorp Vault Enterprise deployment 

### Disconnected requirements
Performing a disconnected deployment, the container image will need to be mirrored within the organization container registry. Here is an example using a Google Cloud Platform private container registry:

```
docker pull ghcr.io/ondat/trousseau:v1.1.3
docker pull vault:1.9.4
docker tag ghcr.io/ondat/trousseau:v1.1.3 us-central1-docker.pkg.dev/shaped-complex-318513/ondat/trousseau:v1.1.3
docker tag vault:1.9.4 us-central1-docker.pkg.dev/shaped-complex-318513/vault/vault:1.9.4
docker push us-central1-docker.pkg.dev/shaped-complex-318513/ondat/trousseau:v1.1.3
docker push us-central1-docker.pkg.dev/shaped-complex-318513/vault/vault:1.9.4
```

During the installation steps, a DaemonSet will be used to deploy Trousseau on Kubernetes. Along with other parameters, the DaemonSet defines what container images are required and where to get them via the paramters ```image:```. 

Based on the above push of the images within a local container image registry, edit the followings in the DaemonSet to target local container image registry:

!!! example "Connected installation"
    ```YAML
        initContainers:
            - name: vault-agent
            image: vault
    ```

    ```YAML
        containers:
            - name: vault-kms-provider
            image: ghcr.io/ondat/trousseau:v1.1.3
    ```

!!! example "Disconnected installation"
    ```YAML
        initContainers:
            - name: vault-agent
            image: us-central1-docker.pkg.dev/shaped-complex-318513/vault/vault:1.9.4
    ```

    ```YAML
        containers:
            - name: vault-kms-provider
            image: us-central1-docker.pkg.dev/shaped-complex-318513/ondat/trousseau:v1.1.3
    ```

## Setup a dev/test HashiCorp Vault

!!! warning 
    This HashiCorp Vault deployment is sufficent for a dev/test environment.   
    Latest [Getting Started](https://learn.hashicorp.com/tutorials/vault/getting-started-dev-server?in=vault/getting-started).

Start a dev/test Vault instance
```bash
vault server -dev -dev-root-token-id=trousseau-demo -dev-listen-address 0.0.0.0:8200 -log-level=debug
```

Expected output of the console from starting HashiCorp Vault
```
2022-03-05T10:35:10.760Z [INFO]  core.cluster-listener.tcp: starting listener: listener_address=0.0.0.0:8201
2022-03-05T10:35:10.760Z [INFO]  core.cluster-listener: serving cluster requests: cluster_listen_address=[::]:8201
2022-03-05T10:35:10.760Z [INFO]  core: post-unseal setup starting
2022-03-05T10:35:10.761Z [INFO]  core: loaded wrapping token key
2022-03-05T10:35:10.761Z [INFO]  core: successfully setup plugin catalog: plugin-directory="\"\""
2022-03-05T10:35:10.762Z [INFO]  core: successfully mounted backend: type=system path=sys/
2022-03-05T10:35:10.762Z [INFO]  core: successfully mounted backend: type=identity path=identity/
2022-03-05T10:35:10.762Z [INFO]  core: successfully mounted backend: type=cubbyhole path=cubbyhole/
2022-03-05T10:35:10.765Z [INFO]  core: successfully enabled credential backend: type=token path=token/
2022-03-05T10:35:10.765Z [INFO]  rollback: starting rollback manager
2022-03-05T10:35:10.765Z [INFO]  core: restoring leases
2022-03-05T10:35:10.765Z [INFO]  expiration: lease restore complete
2022-03-05T10:35:10.765Z [INFO]  identity: entities restored
2022-03-05T10:35:10.765Z [INFO]  identity: groups restored
2022-03-05T10:35:10.766Z [INFO]  core: post-unseal setup complete
2022-03-05T10:35:10.766Z [INFO]  core: vault is unsealed
2022-03-05T10:35:10.769Z [INFO]  core: successful mount: namespace="\"\"" path=secret/ type=kv
2022-03-05T10:35:10.779Z [INFO]  secrets.kv.kv_42530b65: collecting keys to upgrade
2022-03-05T10:35:10.779Z [INFO]  secrets.kv.kv_42530b65: done collecting keys: num_keys=1
2022-03-05T10:35:10.779Z [INFO]  secrets.kv.kv_42530b65: upgrading keys finished
WARNING! dev mode is enabled! In this mode, Vault runs entirely in-memory
and starts unsealed with a single unseal key. The root token is already
authenticated to the CLI, so you can immediately begin using Vault.

You may need to set the following environment variable:

    $ export VAULT_ADDR='http://0.0.0.0:8200'

The unseal key and root token are displayed below in case you want to
seal/unseal the Vault or re-authenticate.

Unseal Key: 2c8NWfkyJg+J5Xg8xauuqlFVIM9XVI/R+/IG/YaBcEs=
Root Token: trousseau-demo

Development mode should NOT be used in production installations!
```

Export the environment variables to setup Vault
```bash
export VAULT_NAMESPACE=admin
export VAULT_TOKEN="trousseau-demo"
```

Verify access to Vault instance
```bash
vault status 
```

```bash
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.9.3
Storage Type    inmem
Cluster Name    vault-cluster-caa9ea4c
Cluster ID      701cb42f-adfc-52aa-2a5a-53776fed0138
HA Enabled      false
```

## Deploying Trousseau

This guide assumes the API Vault endpoint is ```tdevhvc-01.trousseau.io```.

### Pre-deployment Secret

* Create a secret
```
kubectl create secret generic secret-pre-deploy -n default --from-literal=mykey=mydata
```

### Kubernetes ServiceAccount

* Create ServiceAccount
```
kubectl -n kube-system create serviceaccount ```trousseau-auth```
```

* Create the ServiceAccount config file ```trousseau-auth-serviceaccount.yaml```
```YAML
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
   name: role-tokenreview-binding
   namespace: kube-system
roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: kube-system
``` 

* Apply the ServiceAccount config file ```trousseau-auth-serviceaccount.yaml```
```
kubectl apply -f trousseau-auth-serviceaccount.yaml
```

