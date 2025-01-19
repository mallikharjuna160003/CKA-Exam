
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







