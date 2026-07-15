## Kubernetes Persistent Volumes (PV) and Persistent Volume Claims (PVC) with Dynamic Volumes (EBS)

A volume is a storage location attached to a Pod.

   +----------------------+
|          Pod             | 
|                          |
|  +------------------+    |
| |   Container        |   |  
|  +------------------+    |
|         |                |
|         | Mount          |
|         V                |
| +------------------+     |
| |     Volume        |    |
| +------------------+     |
  +----------------------+
A volume lives as long as the Pod exists.

1. emptyDir Volume
Created when Pod starts.
Deleted when Pod is removed.
Example

apiVersion: v1
kind: Pod
metadata:
  name: testpod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: myvolume

  volumes:
  - name: myvolume
    emptyDir: {}
Real-time Scenario
Pod with:
	• Main container writes logs 
	• Sidecar container reads logs 

emptyDir
    |
    +---- Shared between containers

2. hostPath Volume
Mounts a directory from Kubernetes node.

apiVersion: v1
kind: Pod
metadata:
  name: testpod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: myvolume   
  volumes:
- name: host-volume
  hostPath:
    path: /data
  
Example

volumeMounts:
- mountPath: /appdata
  name: host-volume
Real-time Scenario

Node has:

/data/app
Container accesses same directory.

Node
 |
 +-- /data/app
        |
        +-- Mounted into Pod

Drawback
If Pod moves to another node:

Node1 → Data Exists

Node2 → Data Missing
Not recommended for production.


## Kubernetes Persistent Volumes (PV), Persistent Volume Claims (PVC), and Dynamic provisioning using AWS Elastic Block Store (EBS).

Persistent Volume (PV)
A Persistent Volume is a storage resource in a Kubernetes cluster that provides persistent storage, independent of Pod lifecycles. It is defined and managed by the cluster administrator.

Persistent Volume Claim (PVC)
A Persistent Volume Claim is a request for storage by a user. Pods use PVCs to access PVs.
Dynamic Provisioning

Dynamic provisioning automatically creates PVs based on a PVC when a StorageClass is specified. This is particularly useful for cloud-based storage systems like AWS EBS.

What is Dynamic Provisioning?
Dynamic provisioning automatically creates storage volumes when a PVC (PersistentVolumeClaim) is created.
Without dynamic provisioning:



     Admin creates PV manually
      ↓
     User creates PVC
      ↓
    PVC binds to existing PV

With dynamic provisioning:

     User creates PVC
      ↓
     StorageClass triggers provisioner
      ↓
    PV created automatically
      ↓
PVC binds to new PV


## 1. Practical Steps
Pod
 ↓
PVC (Storage Request)
 ↓
PV (Actual Storage)
 ↓
Host Path (/data/k8s on Node)

Step 1: Create Persistent Volume
Create pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/k8s

Apply:

kubectl apply -f pv.yaml
Verify:

kubectl get pv
Expected:

NAME    CAPACITY   ACCESS          MODES       STATUS      CLAIM
my-pv           1Gi        RWO            Available


Step 2: Create Persistent Volume Claim
Create pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi


Apply:
kubectl apply -f pvc.yaml

Verify:
kubectl get pvc
Expected:

NAME     STATUS   VOLUME
my-pvc   Bound    my-pv

Also check PV:
kubectl get pv

Expected:
NAME    STATUS   CLAIM
my-pv   Bound    default/my-pvc

Step 3: Create Pod Using PVC
Create pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc

Apply:
kubectl apply -f pod.yaml
Verify:
kubectl get pods

Step 4: Write Data Inside Pod
Enter Pod:
kubectl exec -it nginx-pod -- bash

Create file:
echo "Hello Kubernetes PV" > /usr/share/nginx/html/index.html

Exit:

exit

Step 5: Verify Data on Node
Login to worker node where pod is running.
Check:

sudo ls /data/k8s
Expected:

index.html
View content:

cat /data/k8s/index.html
Output:

Hello Kubernetes PV

Step 6: Delete Pod

kubectl delete pod nginx-pod
Check file still exists:

cat /data/k8s/index.html
Output:

Hello Kubernetes PV
The pod is deleted, but the data remains because it is stored in the PV.

Important Commands
Check PV:  kubectl get pv
Check PVC: kubectl get pvc
Describe PV:  kubectl describe pv my-pv
Describe PVC: kubectl describe pvc my-pvc
Check mounted volumes: kubectl describe pod nginx-pod

Why do we need PVC?
Without PVC:

Pod → Directly uses PV
This is not flexible.
With PVC:

Pod → PVC → PV
The pod only requests storage; Kubernetes automatically binds it to a matching PV.



2. Practical Steps
Step 1: Create a StorageClass for AWS EBS

Manifest File
Save the following content in a file named storageclass.yaml:

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4


Apply the StorageClass

Run the following command to create the StorageClass:
kubectl apply -f storageclass.yaml
Verify the StorageClass
kubectl get storageclass

Step 2: Create a Persistent Volume Claim (PVC)
Manifest File
Save the following content in a file named pvc.yaml:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-sc

Apply the PVC
Run the following command to create the PVC:
kubectl apply -f pvc.yaml
Verify the PVC
kubectl get pvc

Step 3: Use the PVC in a Pod
Manifest File
Save the following content in a file named pod-with-pvc.yaml:

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-ebs
spec:
  containers:
  - name: app-container
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: ebs-volume
  volumes:
  - name: ebs-volume
    persistentVolumeClaim:
      claimName: ebs-pvc

Apply the Pod Manifest
kubectl apply -f pod-with-pvc.yaml
Verify the Pod
kubectl get pods

Step 4: Check the Persistent Storage
Log into the Pod and verify the mounted volume:
kubectl exec pod-with-ebs -- ls /data


Access Modes
ReadWriteOnce (RWO)
One node can mount.

Node1   will be in used  
Node2   ✘
Used by:
	• EBS 
	• Azure Disk 

ReadOnlyMany (ROX)
Many nodes can read.

Node1 Read
Node2 Read
Node3 Read

ReadWriteMany (RWX)
Many nodes can read and write.

Node1 Read/Write
Node2 Read/Write
Node3 Read/Write
Used by:
	• EFS 
	• NFS 

Volume Mounts
Volume exists but container cannot use it until mounted.
Example:

volumeMounts:
- name: mydata
  mountPath: /app/data
Meaning:

Volume
   |
   +----> Mounted at /app/data


you can achieve persistent storage in Kubernetes using PV, PVC, and dynamic provisioning with AWS EBS. This setup ensures data persistence beyond the lifecycle of individual Pods and simplifies storage management in your Kubernetes cluster.

Kubernetes Persistent Storage in EKS using EBS CSI Driver
Architecture:

EKS Cluster
     |
     V
StorageClass (EBS CSI Driver)
     |
     V
PVC------- pod------pv
     |
     V
EBS Volume Created Automatically
     |
     V
Mounted to Pod

Step 1: Verify EBS CSI Driver
Check whether EBS CSI Driver is installed:

kubectl get pods -n kube-system | grep ebs
Expected output:

ebs-csi-controller
ebs-csi-node
Or:

kubectl get addon
Expected:

aws-ebs-csi-driver

Step 2: Check Existing Storage Classes

kubectl get sc
Example output:

NAME            PROVISIONER                DEFAULT
gp2             kubernetes.io/aws-ebs      Yes
gp3             ebs.csi.aws.com                 Yes
If gp3 exists, you can use it directly.


Step 3: Create StorageClass
Create file:

vim ebs-sc.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3

provisioner: ebs.csi.aws.com

volumeBindingMode: WaitForFirstConsumer

parameters:
  type: gp3
  fsType: ext4
allowVolumeExpansion: true


Apply:

kubectl apply -f ebs-sc.yaml
Verify:

kubectl get sc

Step 4: Create PVC
Create file:

vim pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ebs-gp3
  resources:
    requests:
      storage: 5Gi

Apply:

kubectl apply -f pvc.yaml
Verify:

kubectl get pvc

Output:

NAME        STATUS
nginx-pvc   Pending
This is normal because no Pod is using it yet.

Step 5: Create Deployment
Create file:

vim nginx-deploy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: web-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: web-storage
        persistentVolumeClaim:
          claimName: nginx-pvc

Apply:

kubectl apply -f nginx-deploy.yaml

Step 6: Verify PVC Binding
Check PVC:

kubectl get pvc
Expected:

NAME        STATUS
nginx-pvc   Bound
Check PV:

kubectl get pv
Expected:

NAME                                       CAPACITY
pvc-xxxxxxxx                              5Gi
Notice:
	• Kubernetes automatically created the PV. 
	• You never created the PV manually. 

Step 7: Verify EBS Volume in AWS
Find PV:

kubectl get pv
Describe:

kubectl describe pv pvc-xxxxxxxx
You will see:

VolumeHandle:
vol-0abc123456789xyz
Go to:

AWS Console
   |
   └── EC2
         |
         └── Elastic Block Store
                 |
                 └── Volumes
You will see:

vol-0abc123456789xyz
created automatically.

Step 8: Verify Mount Inside Pod
Get Pod:

kubectl get pods
Enter Pod:

kubectl exec -it <pod-name> -- bash
Check mounted storage:

df -h
Example:

/dev/nvme1n1   5G
Create file:

echo "Welcome to EKS Storage" > /usr/share/nginx/html/index.html
Verify:

cat /usr/share/nginx/html/index.html

Step 9: Delete Pod and Test Persistence
Delete Pod:

kubectl delete pod <pod-name>
Deployment creates new Pod automatically.
Access new Pod:

kubectl exec -it <new-pod> -- bash
Verify:

cat /usr/share/nginx/html/index.html
Output:

Welcome to EKS Storage
Data still exists because it is stored on EBS.

Step 10: Expose Application
Create Service:

kubectl expose deployment nginx \
--type=NodePort \
--port=80
Verify:

kubectl get svc

Useful Verification Commands
Check StorageClass:

kubectl get sc
Check PVC:

kubectl get pvc
Check PV:

kubectl get pv
Describe PVC:

kubectl describe pvc nginx-pvc
Describe PV:

kubectl describe pv <pv-name>
Check mounted volume:

kubectl exec -it <pod-name> -- df -h


Notes 
	• StorageClass: Defines how storage is provisioned dynamically.
	• AccessModes: Determines how a volume can be accessed (e.g., ReadWriteOnce).
	• Dynamic vs Static Provisioning: Dynamic provisioning creates PVs on demand, while static provisioning uses pre-defined PVs.
	• AWS EBS: Ensure your nodes have IAM roles with permissions to create EBS volumes.

Storage Class:

Step 1: Check addon status
Run:

eksctl get addon --cluster my-cluster
Expected:

NAME                         STATUS
aws-ebs-csi-driver    ACTIVE

Step 2: Check EBS CSI pods
Run:

kubectl get pods -n kube-system | grep ebs
You should see:

ebs-csi-controller-xxxxx   Running
ebs-csi-node-xxxxx         Running

Step 3: Enable OIDC provider (recommended)
Run:

eksctl utils associate-iam-oidc-provider \
--cluster my-cluster \
--region ap-south-2 \
--approve

Verify:

aws eks describe-cluster \
--name my-cluster \
--region ap-south-2 \
--query "cluster.identity.oidc.issuer"

You should get something like:

https://oidc.eks.ap-south-1.amazonaws.com/id/xxxxx

Step 4: Create IAM role for EBS CSI Driver
Create service account:

eksctl create iamserviceaccount \
--cluster my-cluster \
--namespace kube-system \
--region  ap-south-2\
--name ebs-csi-controller-sa \
--role-name AmazonEKS_EBS_CSI_DriverRole \
--attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
--approve

Step 5: Update addon with IAM role
Get your AWS account ID:

aws sts get-caller-identity

Example:

{
 "Account": "123456789012"
}
Then:

eksctl update addon \
--cluster my-cluster \ 
--name aws-ebs-csi-driver \
--region ap-south-2 \
--service-account-role-arn arn:aws:iam::502867101594:role/AmazonEKS_EBS_CSI_DriverRole \
--force


Enable OIDC:

kubectl get pods -n kube-system | grep ebs


eksctl utils associate-iam-oidc-provider \
--cluster my-cluster \
--region ap-south-2 \
--approve

Create IAM role:

eksctl create iamserviceaccount \
--cluster my-cluster \
--region ap-south-2 \
--namespace kube-system \
--name ebs-csi-controller-sa \
--role-name AmazonEKS_EBS_CSI_DriverRole \
--attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
--approve

Then install addon:

eksctl create addon \
--cluster my-cluster \
--region ap-south-2 \
--name aws-ebs-csi-driver \
--force


First run this and share output:

aws eks describe-addon \
--cluster-name my-cluster \
--region ap-south-2 \
--addon-name aws-ebs-csi-driver \
--query addon.status


EBS CSI Driver is running,
StorageClass → PVC → Pod
Kubernetes will automatically create an AWS EBS volume.


1. Verify EBS CSI Driver
First confirm:

kubectl get pods -n kube-system | grep ebs
Expected:

ebs-csi-controller-xxxxx   Running
ebs-csi-node-xxxxx         Running

Step 2: Create StorageClass

Create file:

vi ebs-gp3-storageclass.yaml
Add:

apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: ebs-gp3

provisioner: ebs.csi.aws.com

volumeBindingMode: WaitForFirstConsumer

parameters:
  type: gp3
  encrypted: "true"

Apply:

kubectl apply -f ebs-gp3-storageclass.yaml

Check:

kubectl get storageclass

Output:

NAME       PROVISIONER
ebs-gp3    ebs.csi.aws.com

Step 3: Create PVC
Create:

vi ebs-pvc.yaml
Add:

apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: ebs-pvc

spec:
  accessModes:
  - ReadWriteOnce

  storageClassName: ebs-gp3

  resources:
    requests:
      storage: 5Gi

Apply:

kubectl apply -f ebs-pvc.yaml
Check:

kubectl get pvc

You will initially see:

NAME      STATUS
ebs-pvc   Pending

This is normal because:

volumeBindingMode: WaitForFirstConsumer

means EBS will be created only after a Pod uses this PVC.

Step 4: Create Pod using PVC
Create:

vi ebs-test-pod.yaml
Add:

apiVersion: v1
kind: Pod
metadata:
  name: ebs-test-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: ebs-storage
      mountPath: /data
  volumes:
  - name: ebs-storage
    persistentVolumeClaim:
      claimName: ebs-pvc

Apply:

kubectl apply -f ebs-test-pod.yaml

Step 5: Verify PVC
Now:

kubectl get pvc
Expected:

NAME      STATUS   VOLUME
ebs-pvc   Bound    pvc-xxxxx

Step 6: Check Persistent Volume

kubectl get pv
Expected:

NAME                                      STATUS
pvc-xxxxx                                 Bound


If you want to keep the EBS volume
Use:

reclaimPolicy: Retain
Create StorageClass:

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3-retain
provisioner: ebs.csi.aws.com
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3
  encrypted: "true"


Retain behavior

With Retain:

Delete PVC
     |
     ↓
PVC removed
     |
     ↓
PV remains (Released)
     |
     ↓
AWS EBS volume remains

You must manually clean the EBS volume.

Which one to use?
Environment
Reclaim Policy
Testing/Lab
Delete
Development
Delete
Production databases
Retain
Important data
Retain
For your EKS EBS practical, Delete status is fine. For production MySQL/PostgreSQL workloads, usually use Retain status.






