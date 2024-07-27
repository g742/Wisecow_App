# Wisecow_App
1 - Clone the repository:
git clone https://github.com/nyrahul/wisecow.git

2 - Create Dockerfile:
vi Dockerfile ===
FROM python:3.8

WORKDIR /app

COPY . /app

RUN apt-get update && \
    apt-get install -y fortune-mod cowsay && \
    chmod +x wisecow.sh

EXPOSE 4499

CMD ["./wisecow.sh"]

3 - Build docker image by Dockerfile:
docker build -t golu .                # golu is image name given by me

=============================================================================================================================================================================CREATE YAML FILE FOR K8 DEPLOYMENT 
golushbz is my dockerhub ID and goku tag is given by me.
docker tag golu golushbz/goku
docker push golushbz/goku

vi wisecow.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
  labels:
    app: wisecow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: golushbz/goku:latest
        ports:
        - containerPort: 4499
  =========================================================================================================================================================================
  CREATE SERVICE
  vi service.yml

  apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
  - protocol: TCP
    port: 4499
    targetPort: 4499
  type: LoadBalancer
=============================================================================================================================================================================

RUN yml file

kubectl apply -f wisecow.yaml
kubectl apply -f service.yaml

==========================================================================================================================================================================


GITHUB ACTION FOR CICD

.github/workflows/ci-cd.yml


name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    - name: Login to Container Registry
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - name: Build Docker image
      run: docker build -t golu .
    - name: docker push image
      run: docker push golushbz/goku
    - name: Deploy to Kubernetes
      run: kubectl apply -f wisecow.yml
      env:
        KUBECONFIG: ${{ secrets.KUBE_CONFIG_DATA }}
============================================================================================================================================================================

To make comunnication secure. I use TLS certificate
vi ingress.yml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-app-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - wisecow
    secretName: wisecow-tls
  rules:
  - host: wisecow
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-app
            port:
              number: 4499
  
=======================================================================================================================================================================
 AAPPLY TLS
 kubectl apply -f ingress.yml
 INSTALL KCERT as well
        
        




