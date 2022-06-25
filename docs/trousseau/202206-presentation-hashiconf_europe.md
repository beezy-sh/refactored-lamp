
| Type  | Event | Date | Location |
|-------|-------|------|----------|
| in-person conference | [HashiConf Europe](https://www.meetup.com/london-hashicorp-user-group/events/285864430/) | 22/06/2022 | Westerliefde in Amsterdam |

## Abstract
Kubernetes is dominating cloud native application deployment along with the re-platforming and refactoring of legacy applications with the support of HashiCorp Vault. 
However, critical low level Kubernetes components like CNI, CSI, Operators are not capable or design to benefits from the power of HashiCorp Vault.   
The usage of Kubernetes KMS provider plugin solves these challenges by leveraging HahiCorp Vault as the One to rule them all enforcing a ZeroTrust security model.

## Content
<!-- 
```mermaid
graph LR
    a[user] --> b[secret.yml];
    b --> c[kube-apiserver];
    subgraph k8s [kubernetes];
    c --> d[etcd];
    end;
```

```mermaid
graph LR
    a[ZeroTrust] --- b[Vault as KMS]
    b --- c[Kubernetes]
```

```mermaid
graph LR 
    a[ZeroTrust Model] --- b[Kubernetes] & c[Trousseau as Plugin] --- d[Vault as KMS]
```

```mermaid
graph LR 
    a[ZeroTrust Model] --- b[kube-apiserver] 
    subgraph k8s [kubernetes]
    b --> c[Trousseau as Plugin] 
        end
    b --- d[Vault as KMS]
    c --- d
```

```mermaid
flowchart LR
    b[secret.yml] --> c[kube-apiserver];
    subgraph k8s [kubernetes];
    c --> d[trousseau];
    end;
    d --> e[Vault];
    e --> d;
    d --> c;
    subgraph k8s [kubernetes];
    c --> f[etcd];
    end;
```

```mermaid
sequenceDiagram 
autonumber
    Emily->>kube-apiserver: base64 secretxyz
    kube-apiserver->>etcd: write base64 key-value 
    etcd->>kube-apiserver: key-value written
    kube-apiserver->>Emily: secret/secretxyz created
```

```mermaid
sequenceDiagram 
autonumber
    Emily->>kube-apiserver: base64 secretxyz
    kube-apiserver->>trousseau: encrypt this payload
    trousseau->>vault: transit encryption engine
    trousseau->>kube-apiserver: encrypted payload
    kube-apiserver->>etcd: write encrypted key-value
    etcd->>kube-apiserver: key-value written
    kube-apiserver->>Emily: secret/secretxyz created
```
 -->
