
# Create cluster - Lec 23
Install Pre-requisites
1. eksctl utility
2. aws cli
3. kubectl
4. containerd
- cluster name: eksctl-test
- nodegroup name: ng-default
- instance type: t3.micro
- no of nodes: 2
```sh
eksctl create cluster --name eksctl-test --nodegroup-name ng-default --node-type t3.micro --nodes 2
```
check cluster created
```sh
kubectl get all
```
## How does kubectl know eksctl created cluster?
# create cluster via yaml file

```yaml
# eksctl-create-ng.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
 name: eksctl-test
 region: us-west-2

nodeGroups:
 - name: ng1-public
   instanceType: t3.micro
   desiredCapacity: 2

managedNodeGroups:
 - name: ng2-managed
   instanceType: t3.micro
   minSize: 1
   maxSize: 3
   desiredCapacity: 2
```
now run to create two node groups one is managed and self managed
```sh
eksctl create nodegroup --config-file=eksctl-create-ng.yaml
```
check the nodegroups
```sh
eksctl get nodegroup --cluster=eksctl-test
```
We can also create cluster with the same file. It will create cluster and nodegroups
```sh
eksctl create cluster --config-file=eksctl-create-ng.yaml
```
Delete cluster so it will delete associated nodegroups too.
```sh
eksctl get cluster
eksctl delete cluster --name=eksctl-test
```

# create deployment - Lec 24
```yaml
# nginx deployment yaml file nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    environment: test
  name: test
spec:
  replicas: 3
  selector:
    matchLabels:
      environment: test
  template:
    metadata:
      labels:
        environment: test
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
```
Now apply deployment
```sh
kubectl apply -f nginx-deploy.yaml
kubectl get pods
# if the no of pods the node can allow is more then the rest aree in pending state until new node created. Bigger the instance more pods can run.
```
# EKS Managed Nodegroupds - Lec 25

![image](https://github.com/user-attachments/assets/16e11ee5-a968-47da-aedb-10190520e2a5)

![image](https://github.com/user-attachments/assets/88170b0f-c2e9-4ac6-841a-ac53b7efd989)

![image](https://github.com/user-attachments/assets/1281b372-21dc-4738-9a84-f166fb7e2407)

![image](https://github.com/user-attachments/assets/a7407d8e-8c0e-49f6-b3ab-821261b49103)

# Demo  - Lec 26
Create cluster with managed nodegroup
```sh
eksctl create cluster --name eksctl-test --nodegroup-name ng-default --node-type t3.micro --nodes 2 --managed
kubectl get all
kubectl apply -f nginx-deployment.yaml
kubectl get all
```
Ignore the below it is required for ingress 
![image](https://github.com/user-attachments/assets/291f676c-1408-4a8c-b723-00fc658e0032)
- Update by single click on the cluster tab for master nodes upgrade also can be done through the eksctl cli.
- Upgrading the nodegroups
```sh
eksctl upgrade nodegroup --name=managed-ng-1 --cluster=managed-cluster
kubectl get nodes
# will be created more ec2 due to pod disruption, AMI update, all pods need to rescheduled to new nodes. It is rolling update.It is free of charge if using aws AMIs
kubectl get pods
```
# helm -Lec 27
![image](https://github.com/user-attachments/assets/c2e53c95-3373-466d-a609-c6598181aee6)

![image](https://github.com/user-attachments/assets/6d626e46-96a7-4d0b-a7b4-6f1a96dcb19b)

![image](https://github.com/user-attachments/assets/9b884427-d43e-4a38-9527-e2d461c3febd)






























