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

# Installing

## Deploying Service Accounts
To deploy the CSI Volume Driver, you must create three service accounts, create three cluster roles, and then bind the cluster roles to the service accounts. Binding the cluster roles authorizes the service accounts. All of these steps are performed by running the ntnx-csi-rbac.yaml file.

The CSI Volume Driver uses these service accounts to perform read, create, update, and delete actions on PersistentVolumeClaims (PVC), PersistentVolumes (PVs), StorageClasses, and events.


```
# git clone git@github.com:georgesouzafarias/nutanix-k8s-csi.git

# cd ./nutanix-k8s-csi/mandatory/
```

Run the Kubernetes command to create ServiceAccounts, ClusterRole, and ClusterRole binding with the ntnx-csi-rbac.yaml file.

`# kubectl create -f ntnx-csi-rbac.yaml`


Fot confirmation that you created the ServiceAccounts successfully.

`# kubectl get serviceaccounts -n kube-system`

The output displays three ServiceAccounts.
```
NAME                     SECRETS   AGE
csi-attacher             1         1m
csi-ntnx-plugin          1         1m
csi-provisioner          1         1m
```

Confirm that you created the ClusterRole successfully.

`# kubectl get clusterrole -n kube-system`

The output displays three instances (a plug-in, and two runners).
```
NAME                            AGE
csi-ntnx-plugin                 1m
external-attacher-runner        1m
external-provisioner-runner     1m
```

Confirm that ClusterRole binding was successful.

`# kubectl get clusterrolebinding -n kube-system`

The output displays three instances, a plug-in, and two roles.
```
NAME                         AGE
csi-attacher-role            1m
csi-ntnx-plugin              1m
csi-provisioner-role         1m
```

# References
https://portal.nutanix.com/#/page/docs/details?targetId=CSI-Volume-Driver-v10:CSI-Volume-Driver-v10