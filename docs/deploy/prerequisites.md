---
id: prerequisites
title: Prerequisites
sidebar_label: Prerequisites
---

## Installation Requirements

- A Kubernetes (from v1.17 to v1.21) cluster with at least three worker nodes.
- Each worker node requires:
  - At least one 100Gb-available-space SSD for the IOMesh journal and cache
  - At least one 100Gb-available-space HDD for the IOMesh datastore
  - A network card of 10GbE or above for the IOMesh storage network
  - At least 100Gb of disk space in the /opt directory on each worker node

## Set Up Worker Node

Follow the steps below to set up each Kubernetes worker node running IOMesh.

### Set Up Open-ISCSI

1. Install open-iscsi by entering the following script.

  <!--DOCUSAURUS_CODE_TABS-->
    <!--RHEL/CentOS-->
    ```shell
    sudo yum install iscsi-initiator-utils -y
    ```
    <!--Ubuntu-->
    ```shell
    sudo apt-get install open-iscsi -y
    ```

  <!--END_DOCUSAURUS_CODE_TABS-->

2. Edit `/etc/iscsi/iscsid.conf` by setting `node.startup` to `manual`.

    ```shell
    sudo sed -i 's/^node.startup = automatic$/node.startup = manual/' /etc/iscsi/iscsid.conf
    ```
    > **_NOTE_: The default value of MaxRecvDataSegmentLength in /etc/iscsi/iscsi.conf is set at 32,768, and the maximum number of PVs is limited to 80,000 in IOMesh. To create PVs more than 80,000 in IOMesh, it is recommended to set the value of MaxRecvDataSegmentLength to 163,840 or above.**

3. Disable SELinux by entering the following script.

    ```shell
    sudo setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    ```

4. Ensure `iscsi_tcp` kernel module is loaded.

    ```shell
    sudo modprobe iscsi_tcp
    sudo bash -c 'echo iscsi_tcp > /etc/modprobe.d/iscsi-tcp.conf'
    ```

5. Start `iscsid` service.

    ```shell
    sudo systemctl enable --now iscsid
    ```

### Set Up Local Metadata Store

IOMesh stores metadata in the local path `/opt/iomesh`. Make sure that there is at least 100GB of available disk space in the /opt directory. 

### Set Up Data Network

To avoid contention on network bandwidth, set up a separate network segment for the IOMesh Cluster. The `dataCIDR` defines IP block for the IOMesh data network. Every worker node running IOMesh should have an interface with an IP address belonging to `dataCIDR`.
