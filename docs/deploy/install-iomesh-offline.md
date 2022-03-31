---
id: install-iomesh-offline
title: Install IOMesh Offline
sidebar_label: Install IOMesh Offline
---
## Prepare Offline Installation Package
1. download [IOMesh offline installation package](https://cm.smartx.com/share?code=09e83f20-45b0-42d4-a9d8-54f7d5075979)

2. decompress offline installation package
    ```shell
    tar -xf  iomesh-offline-v0.11.tgz && cd iomesh-offline
    ```

## Load Images
load IOMesh images on each kubernetes worker node, choosing right image load command based on your container runtime and container manager tools
<!--DOCUSAURUS_CODE_TABS-->

<!--docker-->
```shell
docker load --input ./images/iomesh-offline-images.tar
```
<!--containerd-->
```shell
ctr --namespace k8s.io image import ./images/iomesh-offline-images.tar
```

<!--podman-->
```shell
podman load --input ./images/iomesh-offline-images.tar
```

<!--END_DOCUSAURUS_CODE_TABS-->

## Install IOMesh Using Offline Installation Package

### Install CSI Snapshotter

The [CSI snapshotter](https://github.com/kubernetes-csi/external-snapshotter) is part of Kubernetes implementation of Container Storage Interface (CSI).
Instasll CSI snapshotter to enable Volume Snapshot feature.

> **_NOTE_: CSI Snapshotter should be installed once per cluster**

1. Install Snapshot CRDs:

    ```shell
    kubectl create -f ./charts/external-snapshotter/config/crd
    ```

2. Install snapshot controller:

    ```shell
    kubectl apply -n kube-system -f ./charts/external-snapshotter/deploy/kubernetes/snapshot-controller
    ```

5. Wait until snapshot controller is ready:

    ```shell
    kubectl get sts snapshot-controller -n kube-system
    ```

    ```output
    NAME                  READY   AGE
    snapshot-controller   1/1     32s
   ```

### Install IOMesh

1. Download `iomesh.yaml` with default configurations:

    ```shell
    ./helm show values charts/iomesh > iomesh.yaml
    ```

2. Customize the `iomesh.yaml`

   **(required)** Fill in the `dataCIDR` according to your network:

    ```yaml
    iomesh:
      chunk:
        dataCIDR: "10.234.1.0/24" # change to your own data network CIDR
    ```

   **(required)** For Kubernetes worker node OS is `CentOS8` or `CoreOS`, set `mountIscsiLock` to `true`. Otherwise, set it to `false`:

    ```yaml
    csi-driver:
      driver:
        node:
          driver:
            mountIscsiLock: true
    ```

   **(optional)** If you only want iomesh to use a part of k8s node's disks, 
   configure the specific node's label in the `chunk.podPolicy.affinity`, for
   example:
   
   ```yaml
   iomesh:
     chunk:
       podPolicy:
         affinity:
           nodeAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               nodeSelectorTerms:
               - matchExpressions:
                 - key: kubernetes.io/hostname # specific node label's key
                   operator: In
                   values:
                   - iomesh-worker-0 # specific node label's value
                   - iomesh-worker-1
    ```
    For more information about pod affinity configuration rule, see: [pod affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

3. Install IOMesh Cluster:

    ```shell
    ./helm install iomesh ./charts/iomesh \
        --create-namespace \
        --namespace iomesh-system \
        --values iomesh.yaml \
        --wait
    ```

    ```output
    NAME: iomesh
    LAST DEPLOYED: Wed Jun 30 16:00:32 2021
    NAMESPACE: iomesh-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    ```

4. You can run `kubectl --namespace iomesh-system get pods` to check out the result:

    ```bash
    kubectl --namespace iomesh-system get pods
    ```

    ```output
    NAME                                                   READY   STATUS    RESTARTS   AGE
    iomesh-chunk-0                                         3/3     Running   0          102s
    iomesh-chunk-1                                         3/3     Running   0          98s
    iomesh-chunk-2                                         3/3     Running   0          94s
    iomesh-csi-driver-controller-plugin-5b66557959-jhshb   6/6     Running   10         3m20s
    iomesh-csi-driver-controller-plugin-5b66557959-p4m9x   6/6     Running   10         3m20s
    iomesh-csi-driver-controller-plugin-5b66557959-w9qbq   6/6     Running   10         3m20s
    iomesh-csi-driver-node-plugin-6pjpn                    3/3     Running   2          3m20s
    iomesh-csi-driver-node-plugin-dj2cd                    3/3     Running   2          3m20s
    iomesh-csi-driver-node-plugin-stbdw                    3/3     Running   2          3m20s
    iomesh-hostpath-provisioner-55j8t                      1/1     Running   0          3m20s
    iomesh-hostpath-provisioner-c7jlz                      1/1     Running   0          3m20s
    iomesh-hostpath-provisioner-jqrsd                      1/1     Running   0          3m20s
    iomesh-iscsi-redirector-675vr                          2/2     Running   1          119s
    iomesh-iscsi-redirector-d2j4m                          2/2     Running   1          119s
    iomesh-iscsi-redirector-sjfjk                          2/2     Running   1          119s
    iomesh-meta-0                                          2/2     Running   0          104s
    iomesh-meta-1                                          2/2     Running   0          104s
    iomesh-meta-2                                          2/2     Running   0          104s
    iomesh-openebs-ndm-569pb                               1/1     Running   0          3m20s
    iomesh-openebs-ndm-9fhln                               1/1     Running   0          3m20s
    iomesh-openebs-ndm-cluster-exporter-68c757948-vkkdz    1/1     Running   0          3m20s
    iomesh-openebs-ndm-m64j5                               1/1     Running   0          3m20s
    iomesh-openebs-ndm-node-exporter-2brc6                 1/1     Running   0          3m20s
    iomesh-openebs-ndm-node-exporter-g97q5                 1/1     Running   0          3m20s
    iomesh-openebs-ndm-node-exporter-kvn88                 1/1     Running   0          3m20s
    iomesh-openebs-ndm-operator-56cfb5d7b6-gwlg9           1/1     Running   0          3m20s
    iomesh-zookeeper-0                                     1/1     Running   0          3m14s
    iomesh-zookeeper-1                                     1/1     Running   0          2m59s
    iomesh-zookeeper-2                                     1/1     Running   0          2m20s
    iomesh-zookeeper-operator-7b5f4b98dc-fxfb6             1/1     Running   0          3m20s
    operator-85877979-5fvvn                                1/1     Running   0          3m20s
    operator-85877979-74rl6                                1/1     Running   0          3m20s
    operator-85877979-cvgcz                                1/1     Running   0          3m20s
    ```

IOMesh Cluster is now installed successfully, please go to [Setup IOMesh](setup-iomesh.md).
