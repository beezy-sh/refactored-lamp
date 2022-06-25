# Trousseau


## Sequence Diagram 

### Kubernetes & Vault Configuration
``` mermaid
sequenceDiagram
participant k8s
participant vault
autonumber
  k8s->>vault: ServiceAccount for Kubernetes Auth
  k8s->>k8s: RBAC rules
  Note right of vault: Enable Transit Engine
  Note right of vault: Create Transit Key
  Note right of vault: Create Transit policy
  Note right of vault: Create Token linked to policy
  Note right of vault: Create Key-Value storing config
  vault->>k8s: Bind vault config to Trousseau Specs
```

### Trousseau Deployment 
``` mermaid
sequenceDiagram
participant k8s
participant trousseau
participant vault
autonumber
  Note over k8s,trousseau: Deployment
  k8s->>trousseau: Apply Trousseau DaemonSet
  Note right of trousseau: Vault Agent Sidecar
  trousseau->>vault: Recover Trousseau ConfigMap config
  vault->>trousseau: Inject Transit Engine Key config
  loop Healthcheck
      trousseau->>vault: validate Vault connection
  end
```
### Trousseau Operations
``` mermaid
sequenceDiagram
participant k8s
participant trousseau
participant vault
autonumber
  Note over k8s,vault: Encryption Operation
  k8s->>trousseau: kube-manager sent Secret for Encryption
  trousseau->>vault: Encrypt Secret payload with Transit Key
  trousseau->>k8s: Send encrypted payload to kube-manager
  Note over k8s,vault: Decryption Operation
  k8s->>trousseau: kube-manager sent Secret for Decryption
  trousseau->>vault: Decrypt Secret payload with Transit Key
  trousseau->>k8s: Send decrypted payload to kube-manager
```

### Vault Token Renewal
``` mermaid
sequenceDiagram
participant k8s
participant trousseau
participant vault
autonumber
  Note over k8s,vault: Token Rotation
  vault->>vault: Renew Token
  Note right of vault: Update Key-Value config
  k8s->>trousseau: Delete all Trousseau POD
  Note right of trousseau: Vault Agent Sidecar
  trousseau->>vault: Recover Trousseau ConfigMap config
  vault->>trousseau: Inject Transit Engine Key config
  loop Healthcheck
      trousseau->>vault: validate Vault connection
  end
```


Dear Bill, 

Renewing the Vault token used by Trousseau to access to the Transit Key Engine is to be considered as a day 2 operation from a vault perspective which requires to have access to the root token or a high privileged token to renew the related token. 

Considering the above, from a security perspective, we can't include this day 2 operation within Trousseau code as this would result in a poor security model and a breach regarding the separation of duties. 

The below steps will address the token renewal day 2 operation.  

Since the max TTL is capped to 768h, the renewal of the token is not possible as it would exceed that parameter. 
To address this renewal cap, a new token needs to be created and inject within the config key-value store. 

On Vault, create a new token
```
[root@tdevhvc-01 trousseau-demo]# vault token create -policy=trousseau-transit-ro
Key                  Value
---                  -----
token                s.Vhc1rXyveyc4Vn8Upd2M1g9H
token_accessor       RTk81akLCLOy2JDyzuAQawDk
token_duration       768h
token_renewable      true
token_policies       ["default" "trousseau-transit-ro"]
identity_policies    []
policies             ["default" "trousseau-transit-ro"]
```

On Vault, check the Trousseau config key-value store:
```
[root@tdevhvc-01 trousseau-demo]# vault kv get /secret/trousseau/config
======= Metadata =======
Key                Value
---                -----
created_time       2022-05-18T00:25:03.155739032Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

========= Data =========
Key               Value
---               -----
transitkeyname    trousseau-kms-vault
ttl               30s
vaultaddress      http://tdevhvc-01.trousseau.io:8200
vaulttoken        s.CkUWvzQamSIiRRzYiY8Jibfy
```

On Vault, update the Trousseau config key-value store:
```
[root@tdevhvc-01 trousseau-demo]# vault kv put /secret/trousseau/config transitkeyname=trousseau-kms-vault ttl=30s vaultaddress=http://tdevhvc-01.trousseau.io:8200 vaulttoken=s.Vhc1rXyveyc4Vn8Upd2M1g9H
Key                Value
---                -----
created_time       2022-06-05T15:31:39.881547821Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            2
```

On Vault, verify the Trousseau config key-value store:
```
[root@tdevhvc-01 trousseau-demo]# vault kv get /secret/trousseau/config
======= Metadata =======
Key                Value
---                -----
created_time       2022-06-05T15:31:39.881547821Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            2

========= Data =========
Key               Value
---               -----
transitkeyname    trousseau-kms-vault
ttl               30s
vaultaddress      http://tdevhvc-01.trousseau.io:8200
vaulttoken        s.Vhc1rXyveyc4Vn8Upd2M1g9H
```

On the Kubernetes cluster, restart the Trousseau DaemonSet Pods:
```
[root@tdevk8s-01 ~]# kubectl rollout restart -n kube-system  daemonset vault-kms-provider
daemonset.apps/vault-kms-provider restarted
```

This last part re-initiate the redeployment of the Pods by starting the Vault Agent to fetch the configuration file which includes the new token. 


