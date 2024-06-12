Lab 1: Kubernetes Operations(kops) on AWS
=============================================================

## Task 1: Launching an EC2 Instance
-----------------------------------------------------------------
Launch an EC2 instance using the Ubuntu Server 22.04 LTS (HVM) AMI, and choose the SSD volume type. Select a t2.micro instance for the machine type.

-----------------------------------------------------------------
## Task 2: Create an IAM role
-----------------------------------------------------------------
Create an IAM role named "kops-admin-role" with the "AdministratorAccess" policy attached.

Now, associate the IAM role "kops-admin-role" with your EC2 instance named "kops" by following these steps:

1. Navigate to the EC2 console and select the "kops" instance.
2. Click on "Action" > "Security" > "Modify IAM role."
3. Search for the "kops-admin-role" role, select it, and click "Update IAM role."


-----------------------------------------------------------------
## Task 3: Setting up a Kubernetes Cluster
-----------------------------------------------------------------
Set the hostname to "kops"
```
sudo hostnamectl set-hostname kops
```

Open a new bash shell
```
bash
```
Download the Kops script
```
wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/kops-v1.25.sh
```
Execute the Kops script
```
. ./kops-v1.25.sh
```
Retrieve information about the existing clusters
```
kops get cluster
```
Get information about the Kubernetes nodes in the cluster
```
kubectl get nodes
```
--------------------------------
Run the following command once you upgrade or restart the cluster
```
kops get cluster
```
```
kops export kubeconfig --admin
```

=================================================EndofLab1=============================================================================

LAB-2  Creating POD
=============================================================
---------------------------------------------------------------
# Task 1 - Create a pod using below Commands
---------------------------------------------------------------
 ```
 kubectl run pod-1 --image nginx --port 80 
```
Check the newly created Pod
```
 kubectl get pod
```
```
 kubectl get pod -o wide
```

Describe Pod using the below command
```
 kubectl describe pod pod-1
```
--------------------------------------------------------------
# Task 2 - Create a pod using below yaml
---------------------------------------------------------------
``` 
vi httpd-pod.yaml
``` 
```
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    env: prod 
    type: front-end
    app: httpd-ws
spec:
  containers:
  - name: httpd-container
    image: httpd
    ports:
       - containerPort: 80
 
 ```
Apply the pod definition yaml
 ```
kubectl create -f httpd-pod.yaml
 ```
 
Check the newly created Pod
 ```
kubectl get pods
 ```
```
kubectl get pods -o wide
 ```
Describe Pod using the below command
 ```
kubectl describe pod httpd-pod
```
=====================================================EndofLab2=========================================================================

