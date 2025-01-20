
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
- eksctl upgrade cluster --name my-cluster --version 1.30 --approve for specific version
- for cluster autoscaler upgrade
```sh
kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=registry.k8s.io/autoscaling/cluster-autoscaler:v1.30
```
while upgrading also Update the Amazon VPC CNI plugin for Kubernetes, CoreDNS, and kube-proxy add-ons. Using commands follow this link 
- <a href="https://repost.aws/knowledge-center/eks-worker-node-actions"> Upgrading the eks cluster</a>
- <a href="https://repost.aws/articles/ARLIq_1BwQQ1iqwkdA3ACy2A/how-to-upgrade-your-eks-cluster-to-the-latest-version"> knowledge article</a>

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


![image](https://github.com/user-attachments/assets/38092591-2e1a-48fe-9fdf-7d1ca97dbb3f)

![image](https://github.com/user-attachments/assets/5ac6f84b-815a-4601-9758-c5cb47f1884d)

![image](https://github.com/user-attachments/assets/307f746f-a2b1-4153-8f09-8de2b370e657)

![image](https://github.com/user-attachments/assets/1bc8b787-6679-465a-b5e1-ce52ad8dcd41)

![image](https://github.com/user-attachments/assets/d8ad3a1f-d9d3-4466-a14d-ea20e283cb28)

# creating autoscaler 

```sh
# cluster-autoscaler.yaml
piVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 20
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 500m
            memory: 256Mi
          limits:
            cpu: 1000m
            memory: 512Mi
  ```
HPA yaml file

```yaml
# hpa.yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```
![image](https://github.com/user-attachments/assets/578863e8-f30e-4469-a9ed-13911c1c3e9b)

create Deployment and Horizontal pod auto scaler also install the metric server 

```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl apply -f nginx-deployment.yaml
kubectl apply -f hpa.yaml
```

![image](https://github.com/user-attachments/assets/193282a2-14dc-40a2-ab28-a1f4803eebc5)

Increase the load 
```sh
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
# Ctrl+C to exit
```
Watch the change in other tab.
```sh
kubectl get deployment php-apache 
kubectl get hpa php-apache --watch # after 5 seconds
kubectl get rs,hpa, deployment, pods
```
This reference to doc <a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/"> HPA</a>

![image](https://github.com/user-attachments/assets/a415c61c-d5a7-4993-9023-e56729ebf57a)

<b>Downscale is slower but scale up is faster.</b>

![image](https://github.com/user-attachments/assets/86df3640-e4ba-4020-8ab5-51159cbb8e43)

![image](https://github.com/user-attachments/assets/7959d5fa-913f-419a-adf2-fd0b8689c92e)










































