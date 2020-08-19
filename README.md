# EKS
![alt text](https://image.slidesharecdn.com/containers-eks-efdba033-6aca-4c87-9222-146570ffd106-234989226-180322152202/95/containers-amazon-eks-1-638.jpg?cb=1521732142)


*Amazon Elastic Kubernetes Service (Amazon EKS) is a fully managed Kubernetes service.*

EKS is the best place to run Kubernetes for several reasons. First, you can choose to run your EKS clusters using AWS Fargate, which is serverless compute for containers. Fargate removes the need to provision and manage servers, lets you specify and pay for resources per application, and improves security through application isolation by design. Second, EKS is deeply integrated with services such as Amazon CloudWatch, Auto Scaling Groups, AWS Identity and Access Management (IAM), and Amazon Virtual Private Cloud (VPC), providing you a seamless experience to monitor, scale, and load-balance your applications. Third, EKS integrates with AWS App Mesh and provides a Kubernetes native experience to consume service mesh features and bring rich observability, traffic controls and security features to applications. Additionally, EKS provides a scalable and highly available control plane that runs across multiple availability zones to eliminate a single point of failure.

EKS runs upstream Kubernetes and is certified Kubernetes conformant so you can leverage all benefits of open source tooling from the community. You can also easily migrate any standard Kubernetes application to EKS without needing to refactor your code.
### Why EKS is the best place to run Kubernetes?
1. You can run EKS clusters using AWS Fargate, which is serverless compute for containers. Fargate removes the need to provision and manage servers, lets you specify and pay for resources per application, and improves security through application isolation by design.
2. EKS is deeply integrated with services such as Amazon CloudWatch, Auto Scaling Groups, AWS Identity and Access Management (IAM), and Amazon Virtual Private Cloud (VPC), providing a seamless experience to monitor, scale, and load-balance your applications.
3. EKS integrates with AWS App Mesh and provides a Kubernetes native experience to consume service mesh features and bring rich observability, traffic controls and security features to applications.
4. EKS provides a scalable and highly-available control plane that runs across multiple availability zones to eliminate a single point of failure.



For making this whole process easy and fast we will use “eksctl” commend. This command provides us automation for deploying our cluster. And this command use cloudformation service of AWS for making our cluster. That is why we need to install ‘eksctl’ software and for using this software command in CLI we must provide the path of this software in environment variables.
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
### To create mysql deployment
```

apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          
          claimName: efs-mysql
```
### For clusterIP service 
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  
  clusterIP: None
```
To run the above code run this command-
```
kubectl create -f mysql.yml
```
### To create wordpress deployment
```

apiVersion: apps/v1 
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: efs-wordpress
```
### For load balancer service-
```

apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  
  type: LoadBalancer
```

To run the above code run this command-
```
kubectl create -f wordpress.yml
```
##  Summary of the task in brief-:


### Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service that makes it easy for you to run Kubernetes on AWS without needing to install, operate, and maintain       your own Kubernetes control plane.

### Steps:

#### 1. Download the AWS CLI program, add it to path variables and configure it to use AWS using the command - aws configure --profile Admin. As terraform will also be using the same credentials for contacting AWS.

#### 2. Write terraform code for deploying a Kubernetes cluster on AWS using EKS.

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
