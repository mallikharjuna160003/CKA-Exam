
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

# creating autoscaler  - Lec 32

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

# Cluster autoscaler 
- Spin the cluster
  
```sh
# create managed nodegroup
# asg-access : creates the auto scaling group and neccesary IAM roles to be accessed by ASG.
# instance type default m5.large
eksctl create cluster --name my-cluster --version 1.30 --managed --asg-access
# cluster auto scaler
kubectl apply -f https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
kubectl edit deployment.apps/cluster-autoscaler -n kube-system
# in the above file update below
# By default, cluster autoscaler will not terminate nodes running pods in the kube-system namespace. You can override this default behaviour by passing in the --skip-nodes-with-system-pods=false flag.
# - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME> 
# - --balance-similar-node-groups
# - --skip-nodes-with-system-pods=false
# cluster auto scaler setup
kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=k8s.gcr.io/cluster-autoscaler:v1.15.6
# to see logs
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
kubectl apply -f cluster-autoscaler-deployment-1.yaml
```
cluster auto scaler deployment yaml file
```yaml
apiVersion: apps/v1
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

for more related to cluster auto-scaler <a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md"> Cluster Auto scaler</a>
![image](https://github.com/user-attachments/assets/4c2fc372-9cbb-4305-bccb-c28c3974eec5)

delete deployment
```sh
kubectl delete deployment php-apache
```
<a href="https://repost.aws/knowledge-center/amazon-eks-troubleshoot-autoscaler"> Troubleshooting autoscaler </a>
# Vertical auto scaler
<a href="https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/quickstart.md"> V Auto Scaler </a>
- Recommended to use the goldilocks visualization tool.
<a href="https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html"> Prometheus setup </a>
<a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-configuration">Create scraper</a>

# Logging in K8s 
- Running Fluentbit as daemonset, as one pod if running on each node agent based.
- creating cloudwatch logs ang log groups to store the pod logs
- create a ConfigMap named cluster-info with the cluster name and the Region to send logs to
- Check the list of log groups in the Region. You should see the following:
1. /aws/containerinsights/Cluster_Name/application
2. /aws/containerinsights/Cluster_Name/host
3. /aws/containerinsights/Cluster_Name/dataplane

![image](https://github.com/user-attachments/assets/95725402-78ef-46ba-b33d-a89c18f32be0)

```sh
# running fluent bit pods
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
# ClusterName=cluster-name
RegionName=cluster-region
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
kubectl create configmap fluent-bit-cluster-info \
--from-literal=cluster.name=${ClusterName} \
--from-literal=http.server=${FluentBitHttpServer} \
--from-literal=http.port=${FluentBitHttpPort} \
--from-literal=read.head=${FluentBitReadFromHead} \
--from-literal=read.tail=${FluentBitReadFromTail} \
--from-literal=logs.region=${RegionName} -n amazon-cloudwatch
# Download and deploy the Fluent Bit daemonset
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml
# verify
kubectl get pods -n amazon-cloudwatch
kubectl logs <pod-name> -n amazon-cloudwatch
# arn for cloudwatch log agent server policy
eksctl create iamserviceaccount --cluster=<clusterName> --name=<serviceAccountName> --namespace=<serviceAccountNamespace> --override-existing-serviceaccounts --attach-policy-arn=<policyARN> --approve
kubectl describe sa fluent-bit -n amazon-cloudwatch
# now check the logs of fluent-bit pod will see same error bcz it has issue so delete and recreate.
kubectl logs <fluent-bit-pod> -n cloud-watch
# kubectl delete daemonset <fluent-bit-pod> -n cloud-watch>
# now check the logs it will show.
```
it will show the below error.
![image](https://github.com/user-attachments/assets/af31667a-ad69-4ffd-b181-e65b263f4062)
Now attach Iam Role Service Account (IRSA) not ec2 instance role bcz the ec2 instance role will give access to all the pods to the cloudwatch logs.So IRSA give only access to fluentbit daemonset.
![image](https://github.com/user-attachments/assets/5307ac5a-6098-42ca-b739-e4871975855a)
See that Annotations have no values.
![image](https://github.com/user-attachments/assets/7e60ea33-3810-4fb2-9353-debf1621f244)
Approve the iam open id connect provider to allow the Service accounts to access the IAM roles attached to pods.
![image](https://github.com/user-attachments/assets/1e3933e2-4499-45bb-a4f0-72a112774434)
Create service account to access the POD iam roles. So the fluentbit daemonset can access the cloudwatch agents
![image](https://github.com/user-attachments/assets/4cc38cbd-e89d-4f86-be4a-8669110b4468)
See the loggroups in cloudwatch.
![image](https://github.com/user-attachments/assets/b0485c4d-bcc7-4e66-b688-2b572e506a86)

# Control Plane Logging.

![image](https://github.com/user-attachments/assets/ceb01c66-0111-4c08-bcf4-290175a26526)

![image](https://github.com/user-attachments/assets/55afc90c-8a3a-42c7-bc38-d24a77aaa370)

![image](https://github.com/user-attachments/assets/1cbedbeb-31b0-4544-85df-968e2312d981)

stream the log groups to Elastic Search via subscription.
![image](https://github.com/user-attachments/assets/00037775-d291-4e9b-bcb2-e8c119396ec1)

# Monitoring using Prometheus - Lec 49

![image](https://github.com/user-attachments/assets/20c7a69d-79c5-415c-83cd-ff69e10e906a)

![image](https://github.com/user-attachments/assets/bb4ac151-d7fc-48a2-b356-1163ed95b834)
<p>
Prometheus discovers and collects metrics from your cluster through a pull-based model called scraping. Scrapers are set up to gather data from your cluster infrastructure and containerized applications. When you turn on the option to send Prometheus metrics, Amazon Managed Service for Prometheus provides a fully managed agentless scraper.
</p>
<p>
You can turn on Prometheus metrics when first creating an Amazon EKS cluster or you can create your own Prometheus scraper for existing clusters. Both of these options are covered by this topic.
</p>
<a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-create">Creating scrapers for K8s cluster using Prometheus</a>
- install kubernetes metric server in cluster
- 
```sh
kubectl create namespace prometheus
# adding repo for prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# Deploy it
helm upgrade -i prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistence.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
 kubectl get pods -n prometheus
 # Port forwarding 
 kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090
 # Choose a metric from the - insert metric at cursor menu, then choose Execute. Choose the Graph tab to show the metric over time. The following image shows container_memory_usage_bytes over time.
 # All of the Kubernetes endpoints that are connected to Prometheus using service discovery are displayed.
# add grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts
mkdir ${HOME}/environment/grafana

cat << EoF > ${HOME}/environment/grafana/grafana.yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
EoF
kubectl create namespace grafana

helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values ${HOME}/environment/grafana/grafana.yaml \
    --set service.type=LoadBalancer
kubectl get all -n grafana
export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo "http://$ELB"
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

```

![image](https://github.com/user-attachments/assets/155b97c9-4b9c-4264-8fe1-2d149a5fd54e)

![image](https://github.com/user-attachments/assets/e24e32ff-4dc2-4512-a342-8a0b0ddc9370)



# Uninstall
```sh
helm uninstall prometheus --namespace prometheus
kubectl delete ns prometheus

helm uninstall grafana --namespace grafana
kubectl delete ns grafana

rm -rf ${HOME}/environment/grafana
```
![image](https://github.com/user-attachments/assets/140f46cd-4068-4266-9bcc-e12e1865aee4)

we can mount the volume to store the and backup the metrics data using mounting volumes.
<a href="https://grafana.com/docs/mimir/latest/configure/configure-metrics-storage-retention/#:~:text=Grafana%20Mimir%20stores%20metrics%20in,older%20than%20the%20configured%20period."> retention period</a>

![image](https://github.com/user-attachments/assets/7b3ba59c-0299-4246-b9e3-ef10bfcbc908)

- in AWS we dont need side car containers to watch the container insights. EKS cluster has the feature.
![image](https://github.com/user-attachments/assets/fa9600d5-de3b-4300-aec4-7a6e7ac56473)

- for the cluster attach the CloudWacthLogsFullAccess policy
![image](https://github.com/user-attachments/assets/04c50417-d712-468f-899f-de56aa1f6d98)

- start reading from <b>To deploy Container Insights using the quick start</b>
<a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html"> container insights</a>

<a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html">Cloudwatch agent setup</a>

![image](https://github.com/user-attachments/assets/8ef1bdd6-9944-43b6-8edb-0891ca4c7ee2)

![image](https://github.com/user-attachments/assets/e527a890-937b-4d0a-b9b8-c308971d2de8)

![image](https://github.com/user-attachments/assets/e3023249-8b61-4d5b-a2cd-b0b38079a322)

```sh
eksctl delete cluster --name=eksctl-test
```
![image](https://github.com/user-attachments/assets/c9e640d0-fd3e-4f59-b2fa-b480407a4f94)

![image](https://github.com/user-attachments/assets/9934cad8-dacc-4265-8146-dd68575db82a)

![image](https://github.com/user-attachments/assets/dfbd7f12-7f8e-4dbd-9242-3f8c5edf712a)

![image](https://github.com/user-attachments/assets/40cb3e27-cfa9-423a-b293-14cc2e5669d4)

![image](https://github.com/user-attachments/assets/2faf6ea4-5357-42a4-bf66-0a536764f79a)




















































