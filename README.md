# EKS
![alt text](https://image.slidesharecdn.com/containers-eks-efdba033-6aca-4c87-9222-146570ffd106-234989226-180322152202/95/containers-amazon-eks-1-638.jpg?cb=1521732142)
### To create EKS Cluster
```

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: ekscluster
  region: ap-south-1
nodeGroups:
  - name: ng1
    desiredCapacity: 2
    instanceType: t2.micro
    ssh:
      publicKeyName: abhishek
  - name: ng2
    desiredCapacity: 2
    instanceType: t2.small
    ssh:
      publicKeyName: abhishek
    ssh:
      publicKeyName: abhishek
```
To create cluster using above code run the command : 
```
eksctl create cluster -f file.yml
```

##  Summary of the task in brief-:


### Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service that makes it easy for you to run Kubernetes on AWS without needing to install, operate, and maintain       your own Kubernetes control plane.

### Steps:

#### 1.Download the AWS CLI program, add it to path variables and configure it to use AWS using the command - aws configure --profile Admin. As terraform will also be using the same credentials for contacting AWS.

#### 2.Write terraform code for deploying a Kubernetes cluster on AWS using EKS.

#### 3.  Inside the cluster resource specify the name of the cluster, VPC inside which the cluster has to be launched, here I have used the ID's of the pre-created default subnet inside the default VPC. But you can also create your own VPC and subnets, define your own rules and use them.

#### 4. After the cluster, define the resource for the node groups - A node group is one or more Amazon EC2 instances that are deployed in an Amazon EC2 Auto Scaling group. All instances in a node group must:

##### Be the same instance type (When we use terraform code to deploy EKS cluster then by default t2.micro instance type is used)
##### Be running the same Amazon Machine Image (AMI)
##### Use the same Amazon EKS worker node IAM role.
#### 5. Now go inside the directory where terraform code is present and run the commands
      terraform init
      terraform apply --aprove
      
##### You can see some services created which is required for cluster like security-groups, load balancer,volumes etc.

##### Now we have to update the config file of Kubernetes to use aws, so that we can use cluster from local system. use command for update file:      
      aws eks update-kubeconfig --name cluster-name
