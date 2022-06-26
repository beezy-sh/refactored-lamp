!!! tip 
    For the latest and greatest news about Discoblocks' releases, please consult [GitHub Releases](https://github.com/ondat/discoblocks/releases).

## **Discoblocks v0** 

### Disco Inferno
!!! note "Release notes - v0.0.4"

    [Git Compare with previous version](https://github.com/ondat/discoblocks/compare/v0.0.3...v0.0.4)
 
!!! info "The current release offers the followings"
    ```
    * Typo fixes in readme and localdev markdowns by @mhmxs
    * Bump github.com/stretchr/testify from 1.7.0 to 1.7.1 by @dependabot
    * Bump github.com/go-logr/logr from 1.2.0 to 1.2.3 by @dependabot
    ```

!!! warning
    This is a pre-release, do not use in production.

!!! example "How to use"

    Prerequisites:

    * Kubernetes cluster
    * Kubeconfig to deploy
    * Configured AWS EBS CSI driver
    * Installed Cert Manager (`make deploy-cert-manager` should help)

    ```console
    cat <<EOF | kubectl apply -f -
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
    name: ebs-sc
    provisioner: ebs.csi.aws.com
    parameters:
    type: gp3
    allowVolumeExpansion: true
    reclaimPolicy: Retain
    volumeBindingMode: WaitForFirstConsumer
    EOF

    kubectl apply -f https://github.com/ondat/discoblocks/releases/download/v0.0.4/discoblocks_v0.0.4.yaml
    ```

    Build your own version
    ``` console
    git clone -b v0.0.4 https://github.com/ondat/discoblocks.git
    cd discoblocks
    export IMG=myregistry/discconfig:current
    make docker-build docker-push deploy
    kubectl apply -f config/samples/discoblocks.ondat.io_v1_diskconfig.yaml
    kubectl apply -f config/samples/pod.yaml
    ```

### Stayin' Alive
!!! note "Release notes - v0.0.3"
    New Contributors

    * @dependabot made their first contribution 
    * @rovandep made their first contribution

    [Git Compare with previous version](https://github.com/ondat/discoblocks/commits/v0.0.3)
 
!!! info "The current release offers the followings"
    ```
    * Initialilze project by @mhmxs
    * Bump github.com/onsi/gomega from 1.17.0 to 1.19.0 by @dependabot
    * Bump k8s.io/klog/v2 from 2.30.0 to 2.60.1 by @dependabot
    * Bump k8s.io/apimachinery from 0.23.0 to 0.23.6 by @dependabot
    * Bump k8s.io/client-go from 0.23.0 to 0.23.6 by @dependabot
    * Bump sigs.k8s.io/controller-runtime from 0.11.0 to 0.11.2 by @dependabot
    * First prototype by @mhmxs
    * Update Dockerfile by @rovandep
    * Add Go Report badge to README by @mhmxs
    * Fix badge links in readme by @mhmxs
    ```

!!! warning
    This is a pre-release, do not use in production.

!!! example "How to use"

    Prerequisites:

    * Kubernetes cluster
    * Kubeconfig to deploy
    * Configured AWS EBS CSI driver
    * Installed Cert Manager (`make deploy-cert-manager` should help)

    ```console
    cat <<EOF | kubectl apply -f -
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
    name: ebs-sc
    provisioner: ebs.csi.aws.com
    parameters:
    type: gp3
    allowVolumeExpansion: true
    reclaimPolicy: Retain
    volumeBindingMode: WaitForFirstConsumer
    EOF

    kubectl apply -f https://github.com/ondat/discoblocks/releases/download/v0.0.3/discoblocks_v0.0.3.yaml
    ```

    Build your own version
    ``` console
    git clone -b v0.0.3 https://github.com/ondat/discoblocks.git
    cd discoblocks
    export IMG=myregistry/discconfig:current
    make docker-build docker-push deploy
    kubectl apply -f config/samples/discoblocks.ondat.io_v1_diskconfig.yaml
    kubectl apply -f config/samples/pod.yaml
    ```