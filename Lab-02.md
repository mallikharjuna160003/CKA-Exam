![image](https://github.com/user-attachments/assets/2b486905-becf-4830-899e-7dea72974353)![image](https://github.com/user-attachments/assets/b2c7fdc2-6a97-44c0-8bfe-5340e39a2c70)![image](https://github.com/user-attachments/assets/6fb5d6de-b205-43b7-905b-b6d93fb0709d)

![image](https://github.com/user-attachments/assets/17f3bb09-b531-4cfc-8807-2b32d8f8be38)

```sh
kubectl create namespace <name-space-name>
```
![image](https://github.com/user-attachments/assets/ebd91545-d74e-4e04-bd3c-94a2de25a448)

![image](https://github.com/user-attachments/assets/f121db25-d06a-4bfb-b3af-7d4d55e35bbd)

![image](https://github.com/user-attachments/assets/b0af0c02-929f-4ea8-8f27-5b8d755a5f3e)

![image](https://github.com/user-attachments/assets/bdb08593-2ab7-4324-9160-84b5dd35825e)

![image](https://github.com/user-attachments/assets/7e3c25bf-15ef-4075-be7d-cfce891919fb)

![image](https://github.com/user-attachments/assets/8e414ba0-3a1b-46bd-af2f-02330fc31aa5)

![image](https://github.com/user-attachments/assets/1a5ad348-5084-4ccb-8924-9b6cfb75d2b4)

![image](https://github.com/user-attachments/assets/89d57e8a-87ee-4de5-97d3-7626af05fcff)

![image](https://github.com/user-attachments/assets/a8abf7eb-97b7-4b53-bd32-24282a2fe076)

![image](https://github.com/user-attachments/assets/b55cbc73-8a41-4e32-ac3a-af1812fb6d69)

![image](https://github.com/user-attachments/assets/79b6535a-0def-414d-9b59-2626b11c6ca6)

![image](https://github.com/user-attachments/assets/65bc9044-980b-40d1-b0ae-bd98b7502593)

<b> NGINX or Traefic does not support the ip mode ingress controller.</b>

![image](https://github.com/user-attachments/assets/017d2e61-c68a-4a73-8bb7-1a0b0f40a739)

![image](https://github.com/user-attachments/assets/8b2cce6f-25be-44d3-8b6e-b1a57d954d93)

![image](https://github.com/user-attachments/assets/cffa6704-e054-48de-986f-8fa1fb27973c)

![image](https://github.com/user-attachments/assets/ca74eddb-aaf4-48a9-a434-9b5f59efb9bc)



<a href="https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/ingress-controller-elastic-kubernetes-services/"> Nginx ingress controller</a>
to setup a demo app follow the steps from the url.
<a href="https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/"> EKS alb ingress controller </a>
Below is the reference for ingress.
```yaml
# service/loadbalancer-aws-elb.yaml
apiVersion: v1
kind: Service
metadata:
 name: nginx-ingress-nlb
 namespace: nginx-ingress
 annotations:
   service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
   service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
   service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
   externalTrafficPolicy: Local
   type: LoadBalancer
   ports:
   - port: 80
     targetPort: 80
     protocol: TCP
     name: http
   - port: 443
     targetPort: 443
     protocol: TCP
     name: https
   selector:
     app: nginx-ingress
```
get pods in nginx namespace
```sh
kubectl get pods -namespace=nginx-ingress
```

<a href="https://docs.aws.amazon.com/eks/latest/userguide/automode.html"> EKS auto</a>

creating ingress controller alb
- tag the subnets to automatically detect the subnets by ingress controller.
- Create a namespace game-2048
- create ingress
- create nodeport 
create all these yaml files and run them.
```yaml
# frontend-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "frontend-deployment"
  namespace: "2048-game"
spec:
  selector:
    matchLabels:
      app: "frontend"
  replicas: 1
  template:
    metadata:
      labels:
        app: "frontend"
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: "frontend"
        ports:
        - containerPort: 80
```

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "frontend-deployment"
  namespace: "2048-game"
spec:
  selector:
    matchLabels:
      app: "frontend"
  replicas: 1
  template:
    metadata:
      labels:
        app: "frontend"
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: "frontend"
        ports:
        - containerPort: 80
```

```yaml
# webserver-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "webserver-ingress"
  namespace: "2048-game"
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
  labels:
    app: webserver-ingress
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "service-frontend"
              servicePort: 80
          - path: /frontend
            backend:
              serviceName: "service-frontend"
              servicePort: 80    
```
![image](https://github.com/user-attachments/assets/eeed05ae-0059-4c03-bbd7-b23fc1404bc8)

![image](https://github.com/user-attachments/assets/b1b0163c-bd2a-4be2-b535-bc0b01f97db2)

Deploying ingress with ip mode.

![image](https://github.com/user-attachments/assets/346462d9-9654-4bdf-b5df-239b321ea37e)

path order matterrs in ingress resource rules.

![image](https://github.com/user-attachments/assets/732135fc-4568-4ab0-a2c6-6f5932005b87)

![image](https://github.com/user-attachments/assets/eaa6ac32-8349-44e9-b531-eb3ac4835a82)

ip mode change and apply. it will make the pod ips as target from load balancer.

![image](https://github.com/user-attachments/assets/a6a8fff0-b47c-4742-83d9-db206ca0dfa4)

![image](https://github.com/user-attachments/assets/70f1d230-336d-4beb-ba32-d0c7d8bbe816)

![image](https://github.com/user-attachments/assets/347f6844-f226-41fa-bec9-04a500242878)

![image](https://github.com/user-attachments/assets/ee938081-b67d-41ce-9898-9240cf32a06a)

![image](https://github.com/user-attachments/assets/a77aa94a-5080-49c1-9101-7acc7f3bf9fd)

![image](https://github.com/user-attachments/assets/22a19ca8-61ac-498f-b13c-43d8c3a9f9fc)

# Demo 3
![image](https://github.com/user-attachments/assets/8dfcf2f1-c17d-4a24-900b-76773c78bcde)

![image](https://github.com/user-attachments/assets/bea3cb44-c7ea-43b1-a249-b1348250e23c)

![image](https://github.com/user-attachments/assets/9a0178a5-1985-48b4-b694-94e026cbd998)

![image](https://github.com/user-attachments/assets/dce0712a-7a0d-4c61-92e1-827f4d17a3e3)

# Canery deployment
![image](https://github.com/user-attachments/assets/65938e92-7da6-45a9-8ca6-8e12e0c9bbf5)

![image](https://github.com/user-attachments/assets/2adb0e6e-dee6-4698-887f-253b69411049)

![image](https://github.com/user-attachments/assets/85208b32-e672-45d0-b82a-81623476a80f)

![image](https://github.com/user-attachments/assets/0335fba7-b86e-4776-92a3-76e9b48a5fbf)

![image](https://github.com/user-attachments/assets/3bd259c4-6076-4d2d-a57f-e3d12cf72360)

![image](https://github.com/user-attachments/assets/a63bd0ad-1a61-4f20-9dd6-c9d94ca0728a)

![image](https://github.com/user-attachments/assets/b4df201d-835b-49d7-9909-a148521c0141)

# CNI
![image](https://github.com/user-attachments/assets/8948901a-6116-4a97-a8e5-235f11b477b8)

![image](https://github.com/user-attachments/assets/38ffec8f-8d61-41bd-a4e8-72278b51961c)

![image](https://github.com/user-attachments/assets/9f1e4823-f9c2-4154-81d9-c8fc4cd1a869)

![image](https://github.com/user-attachments/assets/43b838ae-6903-4d88-8ffc-0042e467b40e)

![image](https://github.com/user-attachments/assets/5856e6eb-65de-492b-a197-dccfaa0c9e7c)

![image](https://github.com/user-attachments/assets/ecff56e6-23c2-4ca8-a5a2-71e3398f65b3)

# Network Policies
![image](https://github.com/user-attachments/assets/78300e79-ffa6-46ab-b41b-3f95f467c6a4)

![image](https://github.com/user-attachments/assets/6a496378-7d1b-4806-8708-7aedd1c1f164)

![image](https://github.com/user-attachments/assets/fca048b1-3f38-4c77-af00-931deb14747b)

![image](https://github.com/user-attachments/assets/eba82a1d-03c5-4cc7-a04e-7b68855b61a9)

![image](https://github.com/user-attachments/assets/87c7ab10-c49a-48a1-8b45-1343c9a8a2d3)

![image](https://github.com/user-attachments/assets/885d38f6-d5fd-4934-901d-86ba64826da8)

Network policy work on layer 3,4 it can see the ip address.It can control traffic at ip level. good for multi tenant cluster. Network Policies are useful for pod security. What we can allow and what not btn the pods communication from namespaces.

![image](https://github.com/user-attachments/assets/bb758aa4-7eab-46dc-9b84-67628574d4f0)

![image](https://github.com/user-attachments/assets/dd98df4c-6f03-454c-8cc2-01ecef319086)

![image](https://github.com/user-attachments/assets/9546fa12-b280-48a8-a8ec-91bc5cee3420)

![image](https://github.com/user-attachments/assets/2b6221c0-8c36-4850-bbc0-a3d11992724f)

![image](https://github.com/user-attachments/assets/d1260810-1ca9-4b08-afa6-4a99d5597e3d)

![image](https://github.com/user-attachments/assets/870a0e8a-4c32-45a9-b1aa-d1af2378c19d)

![image](https://github.com/user-attachments/assets/83cc6be6-e0b2-4c49-84f4-be44c85951a8)

![image](https://github.com/user-attachments/assets/608ac991-6211-4b82-8d8d-7892ee7248c2)

![image](https://github.com/user-attachments/assets/33100eff-97c9-4669-9523-f10273db8870)

![image](https://github.com/user-attachments/assets/0eabcfa0-c6c6-4b0d-b8fa-790ce6ce3f08)

![image](https://github.com/user-attachments/assets/65adb15a-5cac-4d9b-86a0-cbbf4e25fff2)

![image](https://github.com/user-attachments/assets/f04f3d60-38f9-4e83-94b5-2c79f670d42a)

![image](https://github.com/user-attachments/assets/c15f0748-1809-45b9-bedc-1b49f2712d4d)

![image](https://github.com/user-attachments/assets/d8943ce7-9d43-4f2c-a161-e1ecc8930fac)











