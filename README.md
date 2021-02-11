# pingpong-app built in with node js

## How to start test
- Create Dockerfile

FROM node:12-alpine
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --production
COPY . .
CMD ["yarn", "run", "start"]

- Build Docker Image 
(docker built . -t node-app:v2)

- Push ke Docker Hub 
(docker push baguspermadi/node-app:v2)

-Create Deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
  labels:
    app: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app 
    spec:
      containers:
      - name: node-app
        image: baguspermadi/node-app:v2
        ports:
        - containerPort: 3000

-kubectl create -f node-app-deployment.yaml

Create Service.yaml

apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  selector:
    app: node-app 
  type: NodePort
  ports: 
  - protocol: TCP
    port: 3000
    targetPort: 3000

-kubectl create -f node-app-service.yaml

-Test Open in Browser
http://172.17.144.227:30575/

-Install Ingress Controller
minikube addons enable ingress

- Create Manifest Configmap

Ingress-myservice.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myservice
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: bagus.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: node-app-service
                port:
                  number: 3000

kubectl create -f ingress-myservice.yaml

- add Host and ip to /etc/hosts

172.17.144.227 bagus.info

test Curl or open Web with Hostname

