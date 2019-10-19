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

## Deploying Mandatory Resources
To deploy the CSI Volume Driver, you must create:

- Service accounts, create three cluster roles, and then bind the cluster roles to the service accounts
- StatefulSet and a DaemonSet
 

### Clone the repository
```
# git clone git@github.com:georgesouzafarias/nutanix-k8s-csi.git

# cd ./nutanix-k8s-csi/
```

Run the Kubernetes create command to create ServiceAccounts, ClusterRole, and ClusterRole, StatefulSet and DaemonSet.

`# kubectl create -f mandatory/`


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
Confirm that the deployment files are running.

`kubectl get pods -n kube-system`

The output will display five instances to confirm that the deployment was successful:

```
NAME                                         READY     STATUS    RESTARTS   AGE
csi-attacher-ntnx-plugin-0                   2/2       Running   0          1h
csi-ntnx-plugin-4xqfh                        2/2       Running   0          1h
csi-ntnx-plugin-gt742                        2/2       Running   0          1h
csi-ntnx-plugin-rpk88                        2/2       Running   0          1h
csi-provisioner-ntnx-plugin-0                2/2       Running   0          1h
```

## Create a StorageClass

### Nutanix Volumes

#### Creating a Secret

Create a secret for authentication.

- You must create a nutanix user that will be manager the Volumes

With the user created, run:

`echo -n "<NTNX-IP>:9440:<USER>:<PASSWORD>" | base64`

Where:
- NTNX-IP: The Nutanix IP.
- 9440: Nutanix Service Port, default is 9440.
- USER:  Nutanix user.
- PASSWORD: User password.

**Replace the value of KEY field in file storageClass-ntnx-volumes.yaml with the result of the command**
```
apiVersion: v1
kind: Secret
metadata:
 name: ntnx-secret
 namespace: ntnx-storage
 labels:
    app.kubernetes.io/name: ntnx-plugin
    app.kubernetes.io/part-of: csi-ntnx-plugin
data:
 # base64 encoded prism-ip:prism-port:admin:password.
 # E.g.: echo -n "<NTNX-IP>:9440:<USER>:<PASSWORD>" | base64
 key: MTAuOC4djddlslkfpenmesJlcm5ldGVzOnRyaWJ1bmFsX2t1YmVybmV0ZXM=++1
```
#### Managing Storage

After that, you must reclace the follows fields:

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
   name: volume-ntnx
   labels:
    app.kubernetes.io/name: ntnx-plugin
    app.kubernetes.io/part-of: csi-ntnx-plugin
provisioner: com.nutanix.csi
parameters:   
   csi.storage.k8s.io/provisioner-secret-name:  ntnx-secret
   csi.storage.k8s.io/provisioner-secret-namespace: ntnx-storage   
   csi.storage.k8s.io/node-publish-secret-namespace: ntnx-storage
   csi.storage.k8s.io/node-publish-secret-name: ntnx-secret   
   dataServiceEndPoint: <NTNX-IP>:9440
   storageContainer: <NTNX-VOLUME>
   storageType: NutanixVolumes
   description: "Persistent Volume in Nutanix Environments"   
   csi.storage.k8s.io/fstype: ext4
reclaimPolicy: Retain
```
  - csi.storage.k8s.io/provisioner-secret-name: The secret name.
  - csi.storage.k8s.io/provisioner-secret-namespace: the secret namespace.
  - csi.storage.k8s.io/node-publish-secret-namespace: ntnx-storage
  - csi.storage.k8s.io/node-publish-secret-name: ntnx-secret   
  - dataServiceEndPoint: The nutanix IP and Port **NTNX-IP:9440**
  - storageContainer: Storage container in nutanix.
  - storageType: The storage type.
  - description: A simple description.
  - csi.storage.k8s.io/fstype: The filesystem type.
  - reclaimPolicy: Reclaim policy, < Delete or Retain >.

Afte the configuration run:

`# kubectl create -f storageClass-ntnx-volumes.yaml`

Verify the storage class that you specified

`# kubectl get storageclass`

### Nutanix Files

#### Managing Storage
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
   name: volume-ntnx-nfs
   annotations:
        storageclass.kubernetes.io/is-default-class: "false"
   labels:
    app.kubernetes.io/name: ntnx-plugin
    app.kubernetes.io/part-of: csi-ntnx-plugin
provisioner: com.nutanix.csi
parameters:     
   nfsServer: <NTNX-IP>
   nfsPath: <NTNX-files-export>   
reclaimPolicy: Retain
```
  - nfsServer: The nutanix ip
  - nfsPath: The nutanix files export
  - reclaimPolicy: Reclaim policy, < Delete or Retain >.
  
Afte the configuration run:

`# kubectl create -f storageClass-ntnx-volumes.yaml`

Verify the storage class that you specified

`# kubectl get storageclass`


# Test

For test proposes, We will create a PVC(PersistentVolumeClaim).

## Ntnx-Volume test:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: test-volume-pvc
spec:
   accessModes:
      - ReadWriteOnce
   resources:
      requests:
         storage: 1Gi
   storageClassName: volume-ntnx
```
`kubectl apply -f test/teste-volumes-pvc.yaml`

## Ntnx-Files test:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: test-files-pvc
spec:
   accessModes:
      - ReadWriteOnce
   resources:
      requests:
         storage: 1Gi
   storageClassName: volume-ntnx-nfs
```

`kubectl apply -f test/teste-files-pvc.yaml`

# References
https://portal.nutanix.com/#/page/docs/details?targetId=CSI-Volume-Driver-v10:CSI-Volume-Driver-v10