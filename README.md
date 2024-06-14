Note: At the End of each training day 
Perform the below steps:

Navigate to the auto-scaling groups on AWS -> Click on the Master ASG -> Edit -> Change the desired, minimum and maximum fields to 0
Repeat the same for Node ASG.

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
============================================================
Lab 3: Services in Kubernetes
=============================================================
---------------------------------------------------------------
# Task 1 - Create a file named "httpd-pod.yaml" and add the following YAML for a Pod.
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

## Apply the Pod definition using the command:
```
kubectl apply -f httpd-pod.yaml
```

## Check the newly created Pod
```
kubectl get pods
```
```
kubectl get pods -o wide
```
## Describe Pod using below command
```
kubectl describe pod httpd-pod
```

----------------------------------------------------------------------
# Task 2  Setup ClusterIP service
----------------------------------------------------------------------

## Create a file named "httpd-svc.yaml" and add the following YAML for a ClusterIP Service
```
vi httpd-svc.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP

```
## Apply above definition using below to create a ClusterIP service
```
kubectl apply -f httpd-svc.yaml
```
## Describe the service and verify it has populated the endpoints with IP address matching Pod label
```
kubectl get svc
```
```
kubectl describe svc httpd-svc
```
## Get EndPoint of the service
```
kubectl get ep  
```

------------------------------------------------------------------------------
# Task 3  Setup NodePort Service
------------------------------------------------------------------------------

## Modify the service created in the previous task to type NodePort
```
vi httpd-svc.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

## Apply the changes using below command
```
kubectl apply -f httpd-svc.yaml
```
## View details of the modified service
```
kubectl describe svc httpd-svc
```
## Validate connectivity using External IP on NodePort using below or via browser
```
curl <Node-Public-IP>:NodePort
```
OR
```
curl <Service-IP>:NodePort
```

------------------------------------------------------------------------------------
# Task 4  Setup LoadBalancer Service
------------------------------------------------------------------------------------
## Modify the service created in the previous task to type LoadBalancer 
```
vi httpd-svc.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```
##  Apply the changes using below command
```
kubectl apply -f httpd-svc.yaml
```
## Verify that a new service of type LoadBalancer has been created
```
kubectl get svc
```
```
kubectl describe svc httpd-svc
```
## Access the LoadBalancer on the kops instance or via browser
```
curl <LoadBalancer_DNS>
```


-------------------------------------------------------------------------------
# Task 5 Delete and recreate httpd Pod
-------------------------------------------------------------------------------
## Delete the existing httpd-pod using below
```
kubectl delete -f httpd-pod.yaml
```
## View the service details and notice that the Endpoints field is empty
```
kubectl describe svc httpd-svc
```
## Recreate the httpd Pod and view service details Verify that the endpoints is updated with new Pod IP
```
kubectl apply -f httpd-pod.yaml
```
```
kubectl describe svc httpd-svc
```


--------------------------------------------------------------------------------
# Task 6 Cleanup the resources using below command
----------------------------------------------------------------------------------
## Delete the resources created during the lab:
```
kubectl delete -f httpd-pod.yaml
```
```
kubectl delete -f httpd-svc.yaml
```
=============================================endoflab3==================================================
Lab 4 : Deployment
=============================================================

----------------------------------------------------------------------
# Task 1: Write a Deployment yaml and Apply it
----------------------------------------------------------------------
## Create a file named dep-nginx.yaml using content given below
```
vi dep-nginx.yaml
```
## Paste the following content into the file:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx-dep
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.12.2
        ports:
        - containerPort: 80

```
## Save and exit the editor
## Apply the Deployment yaml created in the previous step
```
kubectl apply -f dep-nginx.yaml
```
## View the objects created by Kubernetes Deployment and ReplicaSet 
```
kubectl get deployments
```
```
kubectl get rs
```
## Access one of the Pods and view nginx version
```
kubectl get pods
```
```
kubectl exec -it <pod_name> -- /bin/bash
```
```
nginx -v
```
```
exit
```

-------------------------------------------------------------------------
 # Task 2: Update the Deployment with a Newer Image
-------------------------------------------------------------------------

## Update the nginx image in Pod using below
```
kubectl set image deployment/nginx-dep nginx-ctr=nginx:1.11
```

## Describe the deployment and see that the old pods are replaced with newer ones
```
kubectl describe deployments
```
## Access one of the Pods and view nginx version
```
kubectl get pods
```
```
kubectl exec -it <pod_name> -- /bin/bash
```
```
nginx -v
```
```
exit
```
-----------------------------------------------------------------------------
# Task 3: Rollback of Deployment 
-----------------------------------------------------------------------------

## View the history of Deployments
```
kubectl rollout history deployment/nginx-dep
```
## Rollback the Deployment done in the previous task
```
kubectl rollout undo deployment/nginx-dep --to-revision=1
```
## View the Replica Set and access one of the Pods and view nginx version
```
kubectl get rs
```
```
kubectl get pods
```
```
kubectl exec -it <pod_name> -- /bin/bash
```
```
nginx -v
```
```
exit
```
------------------------------------------------------------------------------
# Task 4: Scaling of Deployments
------------------------------------------------------------------------------
## View the number of Pod replicas created by the Deployment
```
kubectl get deployments
```
```
kubectl get pods
```
## Scale up the deployment to have 8 Pod replicas
```
kubectl scale deployment nginx-dep --replicas=8
```
## Check the Pods and deployment to and verify that the number of Pod replicas are 8
```
kubectl get deployments
```
```
kubectl get pods
```
## Scale down the deployments to 2 Pod replicas
```
kubectl scale deployment nginx-dep --replicas=2
```
## Check the Pods and deployment to and verify that the number of Pod replicas are down to 2
```
kubectl get deployments
```
```
kubectl get pods
```

-----------------------------------------------------------------------------
# Task 5: Deployment Auto Scaling HPA – Horizontal Pod 
Autoscaling
-----------------------------------------------------------------------------
## Create an HPA (Horizontal Pod Autoscaler):
```
kubectl autoscale deployment nginx-dep --min=3 --max=8 --cpu-percent=70
```
## View the HPA status:
```
kubectl get hpa
```
-----------------------------------------------------------------------------
# Task 6 Cleanup the resources using below command
-----------------------------------------------------------------------------
## Delete the resources created during the lab:
```
kubectl delete -f dep-nginx.yaml
```
============================================endoflab4====================================================

Lab 5: DaemonSet in Kubernetes
=============================================================

# ## Create a file named ds-pod.yaml using content given below
```
vi ds-pod.yaml
 ```
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-ds
  labels:
    app: fluent-ds
spec:
  selector:
    matchLabels:
      app: fluentd-app
  template:
    metadata:
       labels:
           app: fluentd-app
    spec:
      containers:
      - name: fluentd-ctr
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```
## Save and exit the editor
## Apply the yaml definition to create a fluent-ds DaemonSet
```
kubectl apply -f ds-pod.yaml
```
## Check the available daemonsets in kubernetes cluster
```
kubectl get ds fluent-ds
```
## Verify that pods for fluent are created one for each node using DaemonSet
```
kubectl get pods -o wide
```
## Delete the Daemonset created during the lab
```
kubectl delete -f ds-pod.yaml
```
## Verify the pods to find (all) the fluentd pods being deleted from each of the nodes
```
kubectl get pods
```
========================================endoflab5======================================================
Lab 6 - StatefulSet Implementation
=============================================================

# Task1 - Create Stateful Set
----------------------------------------------------------------------

## Download file with yaml defintion for an nginx Stateful Set 
```
wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/nginx-sts.yaml
```

## Create a Stateful Set and  headless service by applying the yaml
```
kubectl apply -f nginx-sts.yaml
```

## Validate the headless service creation
```
kubectl get service nginx-svc
```

## Validate the stateful set creation
```
kubectl get statefulset nginx-sts
```

## Watch the pods getting created in an ordinal index fashion
```
kubectl get pods -w -l app=nginx-sts
```

## Go to each of the pods to see the hostname, as a DNS entry is created for each Pod, the hostname inside the Pod should match the pod-name
```
for i in 0 1; do kubectl exec "nginx-sts-$i" -- sh -c 'hostname'; done
```

## Create a busybox pod to test the ping to one of the Ngninx pods by using DNS name of the pod
```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```
```
 nslookup nginx-sts-0.nginx-svc
```
```
nslookup nginx-sts-1.nginx-svc
```
```
exit
```

## Delete the pods for stateful set
```
kubectl delete pod -l app=nginx-sts
```

## In another window notice the new pods getting created in a proper order
```
kubectl get pod -w -l app=nginx-sts
```

## Verify the hostname in each of the newly created pods, it will match the pod name
```
for i in 0 1; do kubectl exec "nginx-sts-$i" -- sh -c 'hostname'; done
```



# Task 3 - Scaling a Stateful Set
----------------------------------------------------------------------------

## Scale the Stateful Set to 5 replicas using below.
```
kubectl scale sts nginx-sts --replicas=5
```

## Verify the pods getting created in ordinal way
```
kubectl get pods -w -l app=nginx-sts
```
## Verify the PV Claim getting created in ordinal fashion
```
kubectl get pvc -l app=nginx-sts
```
## Edit the stateful Set yam and reduce replicas to 3 
```
kubectl edit sts nginx-sts
```
## Notice that the controller deletes the pods one at a time. It waits for one to completely shut down before going to next
```
kubectl get pods -w -l app=nginx-sts
```

## Verify statefulSet’s PersistentVolumeClaims and verify that are not deleted on scaling down. 
```
kubectl get pvc -l app=nginx-sts
```

# Task 4 - Cleanup the resources using below command 
--------------------------------------------------------------------------------
## Delete a Stateful Set
```
kubectl delete -f nginx-sts.yaml
```
## List all the PV and PVC’s that has been allocated to Statefulset pods and delete them as below.
```
kubectl get pvc
```
```
kubectl delete pvc --all
```




