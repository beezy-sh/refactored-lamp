

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
            - name: trousseau-kms-provider
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
            - name: trousseau-kms-provider
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

## Deploy Trousseau

This guide assumes the API Vault endpoint is ```tdevhvc-01.trousseau.io```.

### Pre-deployment Secret

Create a secret
```
kubectl create secret generic secret-pre-deploy -n default --from-literal=mykey=mydata
```

### Kubernetes ServiceAccount

Create ServiceAccount
```
kubectl -n kube-system create serviceaccount trousseau-auth
```

Create the ServiceAccount file
```YAML title="trousseau-auth-serviceaccount.yaml"
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
  name: trousseau-auth
  namespace: kube-system
``` 

Apply the ServiceAccount config file
```
kubectl apply -f trousseau-auth-serviceaccount.yaml
```

### Setup Vault Kubernetes Auth
Gather the Kubernetes ServiceAccount details to configure the Vault Auth for Kubernetes
```
export VAULT_SA_NAME=$(kubectl -n kube-system get sa trousseau-auth \
    --output jsonpath="{.secrets[*]['name']}")
```

```
export SA_JWT_TOKEN=$(kubectl -n kube-system get secret $VAULT_SA_NAME \
    --output 'go-template={{ .data.token }}' | base64 --decode)
```

```
export SA_CA_CRT=$(kubectl -n kube-system config view --raw --minify --flatten \
    --output 'jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
```

```
export K8S_HOST=$(kubectl config view --raw --minify --flatten \
    --output 'jsonpath={.clusters[].cluster.server}')
```

Enable Kubernetes Auth method in HashiCorp Vault
```
vault auth enable kubernetes
```

Generate the command with the above gathered details

```
echo vault write auth/kubernetes/config token_reviewer_jwt="\"$SA_JWT_TOKEN\"" kubernetes_ca_cert="\"$SA_CA_CRT\"" issuer="\"https://kubernetes.default.svc.cluster.local\"" kubernetes_host="\"$K8S_HOST\""
```

The expected output from the above ```echo``` is the command to run on the Vault CLI to enable the ServiceAccount to successfuly authenticate with Vault:
```
vault write auth/kubernetes/config token_reviewer_jwt="eyJhbGciOiJSUzI1NiIsImtpZCI6ImJoOXRmMk1WaDNTN3R3V1lhZ1ZZTm1DenBXRGR3dXN3ckRuY2prSUxfUDQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2ViudzjZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJ2YXVsdC1hdXRoLXRva2VuLWhqY2IyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InZhdWx0LWF1dGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJhNWFiYjI1Ni1kNjA5LTRiMGUtYmQ4MS00Y2I3N2Q4YTRhOWUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06dmF1bHQtYXV0aCJ9.kY2U--XAReFmcdUO-aLLiebdDsnyViPdkM3godbJGkaual_SZUIUXhjNv4adyb5UIRXsvfz_-pXR4Qnr3nv4XxGO59DTyGmStl01VTnY9knYdxZ1gh5SpezsJ5kmOy5NcsFumyCzQTpBCJKP6dXDqtGvm4XvH1Cs_f08fS9RhcZW7qRtuG2A7qqi64FayMsJlXW7y0qs_fOP92gmgpSDhTPcTzzD2XX5M_fDwj7Y-yMOSQrbaqd2kRqet9XubqcASaKwU_YWZ2BaIqX0IG3fErP1AXCg8JjEdgGkcJqF4aFU7DUMOM-Yh73FBO8Kc2WiiHGfrPMbazaXEoFOSQSuOg" kubernetes_ca_cert="-----BEGIN CERTIFICATE-----
MIIBeTCCAR+gAwIBAgJDRPNKBggqhkjOPQQDAjAkMSIwIAYDVQQDDBlya2UyLXNl
cnZlci1jYUAxNjUxMjUxNjc1MB4XDTIyMDQyOTE3MDExNVoXDTMyMDQyNjE3MDEx
NVowJDEiMCAGA1UEAwwZcmtlMi1zZXJ2ZXItY2FAMTY1MTI1MTY3NTBZMBMGByqG
SM49AgEGCCqGSM49AwEHA0IABKi3hk4RAaE117QT8snLG9sFLe4s0OWLOgh+gOIs
x7WYKi6jp+OWLjs22AbyZV8z29GseRFNGkG6ChH90ANq5oWjQjBAMA4GA1UdDwEB
/wQEAwICpDAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBQp7m1LRyIolZUEKnDm
uEZvo1GjwzAKBggqhkjOPQQDAgNIADBFAiBq0052efxOPwieUB3T6wA5RkVKVdDd
ZAPi0FD2284iDQIhAJWPmpj6FixOe7I8kG0vij8rbN0N/+R8CzTrnNZ5Ioit
-----END CERTIFICATE-----" issuer="https://kubernetes.default.svc.cluster.local" kubernetes_host="https://127.0.0.1:6443"
```

!!! warning 
    * requested the quotes to avoid partial writings of the configuration with Vault 
    * the ```kubernetes_host``` might return ```https://127.0.0.1:6443``` resulting to a failure when Vault will dialback to the Kubernetes cluster to do an API call for the ```tokenreviews```. Export manually the external IP/FQDN Kubernetes API endpoint (like ```export K8S_HOST=https://FQDN:6443```). 
    * the ```issuer``` could be different than ```https://kubernetes.default.svc.cluster.local``` when a Kubernetes cluster was set up with a custom ```service-account-issuer``` (see [Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection)).   
    The discovery can be done following this [HashiCorp article](https://www.vaultproject.io/docs/auth/kubernetes#discovering-the-service-account-issuer).


### Setup Vault Transit engine

Enable and configure a Transit engine (Security team)
```
vault secrets enable transit
vault write -f transit/keys/trousseau-kms
```

Create a Transit access policy 
```
vault policy write trousseau-transit-ro -<<EOF
path "transit/encrypt/trousseau-kms" {
   capabilities = [ "update" ]
}
path "transit/decrypt/trousseau-kms" {
   capabilities = [ "update" ]
}
EOF
```

Create a dedicated Token to access the Transit engine
```
vault token create -policy=trousseau-transit-ro
```

Expected output should be similar to:
```
Key                  Value
---                  -----
token                hvs.CAESILoUyuj8STPYKR4AGhaCJylJbkOkmlXlU8pZukoQKc_bGh4KHGh2cy5vQkpnc2g0RVNFZEpsWTA0SWlSNDBxWDQ
token_accessor       BBTat50bsupNqAQNLTXXRhr7
token_duration       768h
token_renewable      true
token_policies       ["default" "trousseau-transit-ro"]
identity_policies    []
policies             ["default" "trousseau-transit-ro"]
```

Export the Token:
```
export TROUSSEAU_TOKEN="hvs.CAESILoUyuj8STPYKR4AGhaCJylJbkOkmlXlU8pZukoQKc_bGh4KHGh2cy5vQkpnc2g0RVNFZEpsWTA0SWlSNDBxWDQ"
```

Create a key-value policy store for Trousseau 
```
vault policy write trousseau-kv-ro - <<EOF
path "secret/data/trousseau/*" {
    capabilities = ["read", "list"]
}
EOF
```

Inject Trousseau configuration parameters within HashiCorp Vault key-value store 
```
vault kv put /secret/trousseau/config transitkeyname=trousseau-kms \
vaultaddress=$VAULT_ADDR vaulttoken=$TROUSSEAU_TOKEN \
 ttl=30s
```

Create a link between the ServiceAccount, namespace, and policies in Vault
```
vault write auth/kubernetes/role/trousseau \
        bound_service_account_names=trousseau-auth \
        bound_service_account_namespaces=kube-system \
        policies=trousseau-kv-ro \
        ttl=24h
```

### Vault Agent ConfigMap

Create Vault Agent ConfigMap file
```YAML title="vault-configmap.yaml"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: trousseau-vault-agent-config
  namespace: kube-system
data:
  vault-agent-config.hcl: |
    exit_after_auth = true
    pid_file = "/home/vault/pidfile"
    auto_auth {
        method "kubernetes" {
            mount_path = "auth/kubernetes"
            config = {
                role = "trousseau"
            }
        }
        sink "file" {
            config = {
                path = "/home/vault/.vault-token"
            }
        }
    }

    template {
    destination = "/etc/secrets/config.yaml"
    contents = <<EOT
    {{- with secret "secret/data/trousseau/config" }}
    --- 
    provider: vault
    vault:
      keynames:
      - {{ .Data.data.transitkeyname }} 
      address: {{ .Data.data.vaultaddress }} 
      token: {{ .Data.data.vaulttoken }} 
    {{ end }} 
    EOT
    }
```

Apply Trousseau's ConfigMap file ```trousseau-configmap.yaml```
```
kubectl apply -f trousseau-configmap.yaml
```

### Trousseau DaemonSet

Create Trousseau custom DaemonSet file
```YAML title="trousseau-daemonset.yaml"
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: trousseau-kms-provider
  namespace: kube-system
  labels:
    tier: control-plane
    app: trousseau-kms-provider
spec:
  selector:
    matchLabels:
      name: trousseau-kms-provider
  template:
    metadata:
      labels:
        name: trousseau-kms-provider
    spec:
      serviceAccountName: trousseau-auth
      priorityClassName: system-cluster-critical
      hostNetwork: true
      initContainers:
        - name: vault-agent
          image: vault                          # (1)
          securityContext:
            privileged: true
          args:
            - agent
            - -config=/etc/vault/vault-agent-config.hcl
            - -log-level=debug
          env:
            - name: VAULT_ADDR
              value: http://FQDN:8200           # (2)
          volumeMounts:
            - name: config
              mountPath: /etc/vault
            - name: shared-data 
              mountPath: /etc/secrets
      containers:
        - name: trousseau-kms-provider
          image: ghcr.io/ondat/trousseau:v1.1.3 # (3)
          imagePullPolicy: Always
          env:                        
            #- name: VAULT_NAMESPACE            # (4)
            #  value: admin
            - name: VAULT_SKIP_VERIFY           # (5)
              value: "true"           
          args:
            - -v=5
            - --config-file-path=/opt/trousseau/config.yaml
            - --listen-addr=unix:///opt/trousseau-kms/vaultkms.socket                            # [REQUIRED] Version of the key to use
            - --zap-encoder=json
            - --v=3
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
              - ALL
            readOnlyRootFilesystem: true
            runAsUser: 0
          ports:
            - containerPort: 8787
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8787
            failureThreshold: 3
            periodSeconds: 10
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 300m
              memory: 256Mi
          volumeMounts:
            - name: vault-kms
              mountPath: /opt/trousseau-kms
            - name: shared-data
              mountPath: /opt/trousseau/
      volumes:
        - name: trousseau-kms
          hostPath:
            path: /opt/trousseau-kms
        - configMap:
            items:
              - key: vault-agent-config.hcl
                path: vault-agent-config.hcl
            name: trousseau-vault-agent-config
          name: config
        - emptyDir: {}
          name: shared-data
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
        - key: node-role.kubernetes.io/etcd
          operator: Exists
          effect: NoExecute
```

1. :material-information: update the version to the desired version.
2. :material-information: set the Vault endpoint URL up.
3. :material-information: update the version to the desired version.
4. :material-information: uncomment with Vault Enterprise and set a namespace in `value` parameter.
5. :material-information: set to false in production with signed TLS certificate.

Apply Trousseau DaemonSet file ```trousseau-configmap.yaml```
```
kubectl apply -f trousseau-configmap.yaml
```

Verify the Trousseau's deployment status
```
kubectl get pod -n kube-system
```

## Enable KMS Provider Plugin

### Generic 
Create the KMS provider plugin file
```yaml title="trousseau-encryptionconfiguration.yaml"
---
kind: EncryptionConfiguration
apiVersion: apiserver.config.k8s.io/v1
resources:
  - resources:
      - secrets
    providers:
      - kms:
          name: vaultprovider
          endpoint: unix:///opt/vault-kms/vaultkms.socket
          cachesize: 1000
      - identity: {} 
```

### Rancher Kubernetes Engine 2