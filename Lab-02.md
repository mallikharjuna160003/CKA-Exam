![image](https://github.com/user-attachments/assets/6fb5d6de-b205-43b7-905b-b6d93fb0709d)

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


