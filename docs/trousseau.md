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


```log
Vault agent started! Log data will stream in below:Sun, Jun 5 2022 11:52:18 pmSun, Jun 5 2022 11:52:18 pm==> Vault agent configuration:Sun, Jun 5 2022 11:52:18 pmSun, Jun 5 2022 11:52:18 pmCgo: disabledSun, Jun 5 2022 11:52:18 pmLog Level: debugSun, Jun 5 2022 11:52:18 pmVersion: Vault v1.9.4Sun, Jun 5 2022 11:52:18 pmVersion Sha: fcbe948b2542a13ee8036ad07dd8ebf8554f56cbSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] sink.file: creating file sinkSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] sink.file: file sink configured: path=/home/vault/.vault-token mode=-rw-r-----Sun, Jun 5 2022 11:52:18 pmSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] sink.server: starting sink serverSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] template.server: starting template serverSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] (runner) creating new runner (dry: false, once: false)Sun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] auth.handler: starting auth handlerSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.127Z [INFO] auth.handler: authenticatingSun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.128Z [DEBUG] (runner) final config: {"Consul":{"Address":"","Namespace":"","Auth":{"Enabled":false,"Username":"","Password":""},"Retry":{"Attempts":12,"Backoff":250000000,"MaxBackoff":60000000000,"Enabled":true},"SSL":{"CaCert":"","CaPath":"","Cert":"","Enabled":false,"Key":"","ServerName":"","Verify":true},"Token":"","Transport":{"CustomDialer":null,"DialKeepAlive":30000000000,"DialTimeout":30000000000,"DisableKeepAlives":false,"IdleConnTimeout":90000000000,"MaxIdleConns":100,"MaxIdleConnsPerHost":9,"TLSHandshakeTimeout":10000000000}},"Dedup":{"Enabled":false,"MaxStale":2000000000,"Prefix":"consul-template/dedup/","TTL":15000000000,"BlockQueryWaitTime":60000000000},"DefaultDelims":{"Left":null,"Right":null},"Exec":{"Command":"","Enabled":false,"Env":{"Denylist":[],"Custom":[],"Pristine":false,"Allowlist":[]},"KillSignal":2,"KillTimeout":30000000000,"ReloadSignal":null,"Splay":0,"Timeout":0},"KillSignal":2,"LogLevel":"DEBUG","MaxStale":2000000000,"PidFile":"","ReloadSignal":1,"Syslog":{"Enabled":false,"Facility":"LOCAL0","Name":"consul-template"},"Templates":[{"Backup":false,"Command":"","CommandTimeout":30000000000,"Contents":"{{- with secret \"secret/data/trousseau/config\" }}\n--- \nprovider: vault\nvault:\n keynames:\n - {{ .Data.data.transitkeyname }} \n address: {{ .Data.data.vaultaddress }} \n token: {{ .Data.data.vaulttoken }} \n{{ end }} \n","CreateDestDirs":true,"Destination":"/etc/secrets/config.yaml","ErrMissingKey":false,"Exec":{"Command":"","Enabled":false,"Env":{"Denylist":[],"Custom":[],"Pristine":false,"Allowlist":[]},"KillSignal":2,"KillTimeout":30000000000,"ReloadSignal":null,"Splay":0,"Timeout":30000000000},"Perms":0,"Source":"","Wait":{"Enabled":false,"Min":0,"Max":0},"LeftDelim":"","RightDelim":"","FunctionDenylist":[],"SandboxPath":""}],"Vault":{"Address":"https://spk-vault.cloudsams.edb.local:8200","Enabled":true,"Namespace":"","RenewToken":false,"Retry":{"Attempts":12,"Backoff":250000000,"MaxBackoff":60000000000,"Enabled":true},"SSL":{"CaCert":"","CaPath":"","Cert":"","Enabled":true,"Key":"","ServerName":"","Verify":true},"Transport":{"CustomDialer":null,"DialKeepAlive":30000000000,"DialTimeout":30000000000,"DisableKeepAlives":false,"IdleConnTimeout":90000000000,"MaxIdleConns":100,"MaxIdleConnsPerHost":9,"TLSHandshakeTimeout":10000000000},"UnwrapToken":false,"DefaultLeaseDuration":300000000000},"Wait":{"Enabled":false,"Min":0,"Max":0},"Once":false,"BlockQueryWaitTime":60000000000}Sun, Jun 5 2022 11:52:18 pm2022-06-05T15:52:18.128Z [INFO] (runner) creating watcherSun, Jun 5 2022 11:53:18 pm2022-06-05T15:53:18.128Z [ERROR] auth.handler: error authenticating: error="context deadline exceeded" backoff=1sSun, Jun 5 2022 11:53:19 pm2022-06-05T15:53:19.128Z [INFO] auth.handler: authenticatingSun, Jun 5 2022 11:54:19 pm2022-06-05T15:54:19.129Z [ERROR] auth.handler: error authenticating: error="context deadline exceeded" backoff=1.76sSun, Jun 5 2022 11:54:20 pm2022-06-05T15:54:20.889Z [INFO] auth.handler: authenticatingSun, Jun 5 2022 11:55:20 pm2022-06-05T15:55:20.892Z [ERROR] auth.handler: error authenticating: error="context deadline exceeded" backoff=3.17sSun, Jun 5 2022 11:55:24 pm2022-06-05T15:55:24.071Z [INFO] auth.handler: authenticatingSun, Jun 5 2022 11:56:24 pm2022-06-05T15:56:24.071Z [ERROR] auth.handler: error authenticating: error="context deadline exceeded" backoff=5.51sSun, Jun 5 2022 11:56:29 pm2022-06-05T15:56:29.585Z [INFO] auth.handler: authenticating
```