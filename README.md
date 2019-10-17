# Nutanix Kubernetes CSI
Sample how to configure a storage class for Nutanix Volumes or Nutanix Files

# Overview
The **Container Storage Interface (CSI)** Volume Driver for Kubernetes uses Nutanix Volumes and Nutanix Files to provide scalable, persistent storage for stateful applications.

Kubernetes contains an in-tree CSI Volume Plug-In that allows the out-of-tree Nutanix CSI Volume Driver to gain access to containers and provide persistent-volume storage. The plug-in runs in a pod and dynamically provisions requested PersistentVolumes (PVs) using Nutanix Files and Nutanix Volumes storage.

When you use Files for persistent storage, applications on multiple pods can access the same storage and also have the benefit of multi-pod read-and-write access.

# Persistent Storage
Nutanix supports dynamic provisioning of PersistentVolumeClaims (PVC) using the CSI Volume Driver, which runs in a Kubernetes pod. An administrator must configure a storage class for the persistent volume provisioner. The provisioner watches for a PVC request for the configured storage class and then creates a PersistentVolume (PV) for that request.

# Prerequisites
Make sure the following prerequisites are met before deploying the CSI Volume Driver.

- Kubernetes version 1.13 and later.
- AOS 5.6.2.x.
  - (Nutanix Volumes only) Kubernetes worker nodes must have the iSCSI package installed. The package name depends on the Linux flavor.
  - Centos: iscsi-initiator-utils
  - Ubuntu: open-iscsi
(Nutanix Files only) NFS mount packages:
  - nfs-utils for CentOS
  - nfs-common for Ubuntu

# References
https://portal.nutanix.com/#/page/docs/details?targetId=CSI-Volume-Driver-v10:CSI-Volume-Driver-v10