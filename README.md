# AWS_EKS

Description- 
     Part 1- 
Deploy a simple EKS Cluster and deploying our website over it using EBS and ELB services,
    Part 2-
Deploy a Mixed-EKS Cluster using the spot instance and deploying the MySQL and WordPress over it using the ELB, EC2, EFS services. 

Prerequisite-
Basic Knowledge of AWS EC2 instance, EBS, LB, EFS, EKS.
Create an AWS IAM account with Administrator Power.

Software Required-
Install AWS CLI 
(https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html )
Install eksctl
( https://github.com/weaveworks/eksctl/releases )
Install kubectl
( https://kubernetes.io/docs/tasks/tools/install-kubectl/ )

Amazon Elastic Kubernetes (Amazon EKS)- 
Amazon Elastic Kubernetes Service (Amazon EKS) is a fully managed Kubernetes service. 
Serverless Option: You can choose to run your EKS clusters using AWS Fargate, which is a serverless compute for containers. Fargate removes the need to provision and manage servers, lets you specify and pay for resources per application, and improves security through application isolation by design. 
Secure: EKS automatically applies the latest security patches to your cluster control plane.
High Availability: EKS runs the Kubernetes management infrastructure across multiple AWS Availability Zones, automatically detects and replaces unhealthy control plane nodes, and provides on-demand, zero downtime upgrades and patching.
EKS is deeply integrated with services such as Amazon CloudWatch, Auto Scaling Groups, AWS Identity and Access Management (IAM), and Amazon Virtual Private Cloud (VPC), providing you a seamless experience to monitor, scale, and load-balance your applications. 

Now, Let’s Begin- 
 Part 1-  Deploy a simple EKS Cluster and deploying our website over it using EBS and ELB services:

  Configure your AWS through the CLI by running the aws configure cmd. Provide your credentials and name of the region where you want to launch your cluster. 
>>> aws configure
I am launching my cluster in the us-west-2 region.


Create a directory to store all the files for your project. 
For me, the name of the directory is eks_code.

Now, we create a .yml file containing the configuration details of the cluster to be launched.

For launching the cluster, we use, eksctl create cluster -f <filename> cmd. It takes about 15 min for AWS to deploy your cluster. For me, the filename is cluster.yml.
>>> eksctl create cluster -f cluster.yml


Now, you can check the cluster creation using the cmd,
>>> eksctl get cluster


Also, you can check the nodes of the cluster using the cmd,
>>> kubectl get nodes

It is a good practice, to create your own namespace rather than to use the default namespace. You can create a namespace using kubectl create namespace <namespace_name> cmd. For me, the namespace_name is rajshree-ns.
>>> kubectl create namespace rajshree-ns


You can list all the namespace using the cmd,
>>> kubectl get ns


To set the newly created namespace as the current namespace, we use cmd,
>>>  kubectl config set-context --current --namespace=rajshree-ns


 You can check the running pods in the current namespace using,
>>>  kubectl get pods
Currently, no pods are running in this namespace as we haven’t launched any yet.


 You can check where your cluster is running using,
>>>  kubectl cluster-info
This cmd shows the location where your master is running.


 Now, we deploy a container to launch our PHP website, using cmd,    kubectl create deployment <deployment_name> --image=                                  <image_name>. For me, the deplyment_name is myweb and the
  image_ name is vimal13/apache-webserver-php.
You can use any docker image but i am using this image,  vimal13/apache-webserver-php

Once created, the Deployment ensures that the desired number of Pods are running and available at all times. The Deployment automatically replaces Pods that fail or are evicted from their nodes. 

Now, check the pods are successfully running or not in the current   namespace using,
>>>  kubectl get pods
In my case, they are running fine. 


We create a cloud-network Load Balancer. This provides an externally accessible IP address that sends traffic to the correct port on your cluster nodes provided your cluster runs in a supported environment and is configured with the correct cloud load balancer provider package. We can do this using kubectl expose deployment <deployment_name> --type= LoadBalancer --port=80  cmd. For me, the deplyment_name is myweb.
>>> kubectl expose deployment myweb --type=LoadBalancer --port=80


To get the detail about the service, we use,
>>> kubectl get services.
Here, we see one Load Balancer come up that means our pod is exposed.


 You can scale your pods that means you are creating replicas of the pods. We can scale using  kubectl scale deployment <deployment_name> --replicas=<number>. It will create (number-1) more replicas for you because one is already running there. 
>>> kubectl scale deployment <deployment_name> --replicas=4.

 Now, we copy  the external IP address of the load balancer and paste it in the address bar of the web browser. You will see the deployed web page.


 Now, we copy  the external IP from the load balancer and paste it in the address bar of the web browser. You will see the deployed web page.

 To check the pods in detail, we use,


In pods, the storage we use is a non-persistent type, meaning data stored inside will delete on the occurrence of any failure. To make the storage permanent i,e, persistent we mount one volume known as EBS in aws. After mounting to the centralized storage our data becomes permanent.

 Now, we create a .yml file containing the configuration details of the PVC (Persistent Volume Claim). This is a service which is connected to the client-side and if any request is generated from the client then it will contact the PV(persistent volume) and this PV sends a request to the storage class. This storage class is an intelligent program that knows how to provide the storage from different storage groups. 

For launching the pvc, we use, kubectl create -f <filename> cmd. It takes about 2-3 min. For me, the filename is pvc.yml.
>>> kubectl create -f pvc.yml


 You can see your deployment using,
>>>kubectl get deploy


Now, you can check the pvc creation using the cmd,
>>> kubectl get pvc

At the moment, the status of our pvc is pending.

 After creating the PVC, you need to go to your deployment and update deployment because without updating the deployment PV will not be created. Also, update the mount path. 

For editing the deployment we use kubectl edit deploy <deployment _name>. For me, deployment_name is myweb.


You have to add the above code in the spec section of the deployment with proper indentation and save it.

 Your pvc status should be bound. Check your pvc using,
>> kubectl get pvc


Now, you can check the pv creation using the cmd,
>>> kubectl get pv


Now, you can delete your cluster using,
>>> eksctl delete cluster -f cluster.yml

Part 2-  Deploy a Mixed-EKS Cluster using the spot instance and deploying the MySQL and WordPress over it using the ELB, EC2, EFS services:

For launching the cluster, we use, eksctl create cluster -f <filename> cmd. It takes about 15 min for AWS to deploy your cluster. For me, the filename is mycluster.yml.
>>> eksctl create cluster -f mycluster.yml


Update your kubeconfig file, using, aws eks update-kubeconfig --name <cluster_name> cmd. For me, cluster_name is Mix-Cluster.
>> aws eks update-kubeconfig --name Mix-Cluster

It will update all the information about the cluster into the file.

Now we are logging into all the ec2 instances with a root account and installing one software using yum install amazon-efs-utils cmd. Do this step for all the instances which we launch. We login, using,
>>> ssh -i <keyname.pem> -l ec2-user <ip>



Also, you can check the nodes of the cluster using the cmd,
>>> kubectl get nodes


Now, you can check the cluster creation using the cmd,
>>> eksctl get cluster -- name <Cluster-name>


Now we are creating one EFS here. We create EFS Because EFS is centralized storage and whenever we do any changes from any region, they will also reflect in other associated regions. So if any client changes something from region 1a and if he/she again login with 1b region then he/she will get the same edited file.

Now select the create file system option. Select the VPC and change from default to VPC of your cluster and also change the security group from default to clusterShareNodeSecurityGroup. It will allow the nodes to ping each other and also share the file internally.

Now, your EFS is created.


Copy the file system-id. Now we are going to create a  .yml file that helps us to provision this efs into our cluster. Paste the copy system-id into the values section. As I do below. Also, do changes in the server part of your yml file.


 Now, we will provision our efs into our cluster using kubectl create -f <file_name> cmd. For me, file_name is provisioner_name.yml
>>> kubectl create -f provisioner_name.yml 


Also, you can check the pods using the cmd,
>>> kubectl get pods


Now we create one RBAC which helps us to secure our namespace. From the outside world.


To run this yml file we use, kubectl create -f <file_name> cmd. For me, file_name is namespace.yml.
>> kubectl create -f namespace.yml











Now we create one storage class efs. This storage class contacts the efs to provide the storage. And we create one pvc which creates PV and this PV contact the storage class and storage class provide the storage from efs.

To run this yml file we use, kubectl create -f <file_name> cmd. For me, file_name is storage.yml.
>> kubectl create -f storage.yml

A persistent storage of 10-10 GB is created in the EFS.

A storage class is created.

Now, we can check pvc by using,
>>> kubectl get pvc


Also, we can check pv by using,
>>> kubectl get pv


After doing all the setup we are ready to launch our web app and with MySQL database. For launching, we will place all the file in one folder and configure one kustomization (kustomization is one feature of Kubernetes which execute all the files in a single go the only challenge is all file is in the same folder and kustamiztion contain the name of the file which is executed in a single go.) file which launches both my SQL and WordPress.
For me, deploy-mysql is the file for the SQL and deploy-wordpress is the file for the wordpress. And kustomization is the file for the kustomization code.



 You can get all the information using,
>>> kubectl get all


 Now, copy the IP of the wordpress service and paste in your web browser.













 As Wordpress is a paid service, you are recommended to delete the cluster.





