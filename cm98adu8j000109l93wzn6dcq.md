---
title: "Deploying a 3-Tier Application on AWS EKS with a Custom Domain - Step-by-Step Guide for Beginners"
datePublished: Tue Apr 08 2025 09:17:02 GMT+0000 (Coordinated Universal Time)
cuid: cm98adu8j000109l93wzn6dcq
slug: deploying-a-3-tier-application-on-aws-eks-with-a-custom-domain-step-by-step-guide-for-beginners
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1744103721458/7fea4fca-eebf-4d54-a376-76be935567e1.png
tags: cloud, docker, aws, kubernetes, devops, helm, alb, ingress, eks, namecheap, eks-cluster, eks-project

---

## In this blog, I’ll walk you through how I deployed a real-world **three-tier application** (frontend, backend, and database) on **Amazon EKS (Elastic Kubernetes Service) using domain purchased from *Namecheap***

## **using** industry-standard tools like **Docker**, **Kubernetes**, **Ingress**, and **AWS Load Balancer**.

### In today's cloud-native era, deploying scalable applications is crucial. The 3-tier architecture offers modularity and scalability, making it a preferred choice. However, integrating this with AWS EKS and configuring a custom domain presents challenges that we'll address in this guide.

This project helped me understand how production-grade systems are built, deployed, and managed at scale using DevOps practices.

Whether you’re a student like me or a beginner DevOps enthusiast, this guide will give you a clear idea of:

* Designing a three-tier architecture for cloud
    
* Containerizing and deploying apps using Kubernetes
    
* Exposing services securely with ALB + Ingress
    

Using AWS tools like EKS, IAM, and Load Balancer Controllers

---

### Prerequisites:

* To follow along, you should have:
    
    * Basic knowledge of Kubernetes and AWS
        
    * An EKS cluster up and running
        
    * kubectl configured to access your cluster
        
    * AWS CLI configured
        
    * A registered domain (e.g., from Namecheap) \[You can perform on your own ip address as well.\]
        

Let’s dive in. 🛠️

---

## Troubleshooting & Mistakes I Made

### 1\. IAM Role AccessDenied

I initially didn't attach the correct policy to the IAM role used by the AWS Load Balancer Controller. It threw an `AccessDenied: elasticloadbalancing:DescribeListenerAttributes`error.  
**Fix**: Attach it `ElasticLoadBalancingFullAccess` to the controller's IAM role.

### 2\. Ingress Not Working

I forgot to use the correct annotation in the Ingress manifest:

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

---

### How we are going to perform things:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743908925617/6e713ac6-293c-4eb4-ab1f-4f4ea4fd7aa4.png align="center")

---

### Creating An AWS "t2.micro" EC2 Instance

Create an instance and ssh into the instance through your terminal  
After doing ssh into instance, copy the source code from github, i.e., clone the github repo in terminal

Link: [https://github.com/dakshsawhneyy/Three\_Tier\_App.git](https://github.com/dakshsawhneyy/Three_Tier_App.git)

---

### Creating a DockerFile for React in frontend folder

```dockerfile
FROM node:16

WORKDIR app

# copy these files in app folder
COPY package.json package-lock.json ./
RUN npm install

COPY . .

CMD ["npm","start"]
```

### And also install docker in your instance

```bash
sudo apt-get install docker.io
sudo usermod -aG docker $USER && newgrp docker
```

### Now build image using command

```bash
docker build -t frontend-app .
```

## Running the docker image

```bash
# React runs on port 3000
docker run -d -p 3000:3000 frontend-app:latest
```

---

## Configuring AWS CLI with our terminal

Paste this code in our terminal - for installation for aws cli

```bash
cd
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
```

## Configuring AWS CLI

### Create An IAM User on AWS

### Provide full administrator access permission to your user

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743912607144/f79b1a55-5d30-4e3a-9c4f-b5c18e2117a9.png align="center")

### While creating, select CLI for access ID and password

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743912374418/05ec2f98-e0c1-418d-9502-cf162e03b57f.png align="center")

Then you’ll get ID and Password

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743912419811/1914a125-f3ac-4acd-a4b6-5d4180540cac.png align="center")

Now type in terminal

```bash
aws configure
```

And then paste these in your terminal

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744077273567/f75fb34f-a457-4ac9-919e-aa9350ec5db6.png align="center")

### And your AWS is successfully configured with your terminal 🎉🎉

---

## Pushing Image on AWS ECR

Create a repository in ECR

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743912855751/5cbd39d9-72d0-458b-9d46-5ab427f67055.png align="center")

And tap CREATE

We Got our empty repository

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743912925870/830b342c-1314-439f-986b-f7b981d8b48e.png align="center")

Now we're going to push image into this repository

### For pushing, click on “view push commands."

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743913212222/932b6acb-e34c-44cc-9e14-0b358f79c248.png align="center")

And perform all these on your terminal and successfully you have pushed image to AWS ECR🎉🎉

---

## Now, Creating DockerFile for backend

```dockerfile
FROM node:16

WORKDIR app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["node","index.js"]    # Since it is node app - so need to run index.js
```

### Instead of building image again, create a repo on ECR and then directly push it to that

After creating an image from the Dockerfile, push it to ECR by following the steps given in “ VIEW PUSH COMMANDS.”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744077425905/bc766240-d6a7-49a8-88a9-d9451d1d8cff.png align="center")

And you have deployed backend image as well on ECR 🎉🎉

## On local - if you want to see database is running or not

```bash
docker run -d -p 8080:8080 three-tier-backend:latest     # run the image built
docker ps    # see running processes
```

Then see logs on the container running

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744078012879/f4282cb1-c859-434f-8c10-8eee68d65a5a.png align="center")

So it is unable to connect to MongoDB. Therefore we need to make it connect to Database

### To fix this, when we integrate this with K8S, we will fix the issue there with the help of services, which are used to communicate between pods.

---

## Now creating K8S cluster on AWS EKS

installing **eksctl and kubectl** on your terminal

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

```bash
curl -o kubectl https://amazon-eks.s3.ap-south-1.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl    # make this file elecutable
sudo mv ./kubectl /usr/local/bin    # moving this to bin so we dont need to write ./ everytime . i only can write kubectl command 
kubectl version --short --client    # check the version of installed kubectl
```

## Now setting up cluster

```bash
eksctl create cluster --name three-tier-cluster --region ap-south-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster    # binding kubectl with eks cluster - so when i write kubectl get nodes, it actually provide nodes
kubectl get nodes
```

Now wait for 15-20 minutes because creation of cluster takes this much of average required time. 🥲

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744080353027/234b4dd1-f0f7-4eb3-bc61-b37296915fa5.png align="center")

And our cluster got ready when you check in EKS

---

## Making YAML files for MongoDB

Create a mongo Folder for keeping its yaml files inside it

Creating secrets.yaml inside mongo folder for storing mongoDB username and password

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-sec
  namespace: workshop
type: Opaque
data:
  username: cm9vdAo=    # root
  password: cm9vdAo=    # root
```

For username and password, we need to encrypt it using base64 and then paste them here

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744081113981/e3a1fb1f-82ca-4075-96cd-694929cd0a61.png align="center")

So put in the username and password, which is root from here.

## Now creating service for MongoDB

// vim service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-svc
  namespace: workshop
spec:
  selector:
    app: mongo
  ports:
    - name: mongo-svc
      protocol: TCP
      port: 27017
      targetPort: 27017
```

## Now, creating deployment for MongoDB

// vim deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  namespace: workshop
  labels:
    app: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
       - name: mongo
         image: mongo:latest
         ports:
         - containerPort: 27017
         resources:
           requests:
             memory: "512Mi"
             cpu: "250m"
           limits:
             memory: "1Gi"
             cpu: "500m"
         env:
          - name: MONGO_INITDB_ROOT_USERNAME
            valueFrom:
             secretKeyRef:
              name: mongo-sec
              key: username
          - name: MONGO_INITDB_ROOT_PASSWORD
            valueFrom:
             secretKeyRef:
              name: mongo-sec
              key: password
```

### Before applying these, we need to create namespace ‘workshop.’

Write in the terminal “kubectl create namespace workshop,” and it will create namespace “workshop.”

### Then hit terminal with “kubectl apply -f secrets.yaml” and then “kubectl apply -f deploy.yaml.” and “kubectl apply -f service.yaml”

and if you press kubectl get pods -n workshop

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744082188782/8173aee6-a072-41d6-b0a4-ff9d79ea92d5.png align="center")

Pods are running and mongoDB deployment is created. 🎉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744082281910/b22b556a-0e5c-4261-897e-6025f08addf6.png align="center")

---

## Now, creating Backend Deployment and Backend Service

// Creating backend-service.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  labels:
    app: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: # ECR Image
        ports:
        - containerPort: 8080
        env:
          - name: MONGO_URI
            value: mongodb://mongodb-svc:27017/todo?directConnection=true    # provided service to backend so it can connect with database
          - name: MONGO_USERNAME
            valueFrom:
              secretKeyRef:
                name: mongo-sec
                key: username
          - name: MONGO_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mongo-sec
                key: password
```

// vim backend-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: workshop
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

Apply Both Deployment As Well as Service

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744100631516/06d808d8-5470-4b54-87ed-5e954fb3b7be.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744084472847/23cd5915-996d-4a97-b662-0b65a6e305c0.png align="center")

---

## Creating Backend Frontend and FrontendService

// vim frontend-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  workspace: workshop
  labels:
    app: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: # from ECR
        ports:
        - containerPort: 3000
        env:
          - name: REACT_APP_BACKEND_URL
            value: "http://challenge.cctlds.online/api/tasks"    # cctlds.online is my domain name
```

// vim frontend-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
  namespace: workshop
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744094187041/33d8baf9-3513-4d53-a61f-7d9efc4fe576.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744100725046/bab42f6e-3b05-4075-93ae-13001f1b468b.png align="center")

All Are Running

---

## For making it publicly accessible, we need INGRESS CONTROLLER

Our cluster is isolated, basically, so to make it accessible to outside world, we need Ingress Controller

So for installing Ingress Controller, we need to install **HELM** !!

### Install AWS Load Balancer

```yaml
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=ap-south-1
```

### Deploy AWS Load Balancer Controller

```yaml
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=three-tier-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744099217092/e161d3e5-f0d2-409d-8b9a-f8043b56b981.png align="center")

---

## For routing, we need ingress

// vim [full\_stack\_lb.yaml](https://github.com/dakshsawhneyy/Three_Tier_App/blob/master/k8s_manifests/full_stack_lb.yaml)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mainlb
  namespace: workshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  ingressClassName: alb
  rules:
    - host: challenge.cctlds.online
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 3000
```

To apply this file, type “kubectl apply -f fullstack\_lb.yml.”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744099269658/d94eb33c-72b2-4fd5-94ec-3ac6c03cf9cf.png align="center")

---

# Issue occuring in ingress

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744101464561/c8b9c36d-032b-4dc6-8bc4-a8d365123198.png align="center")

The address is not showing in Ingress—we need to debug it.

So run

```bash
kubectl describe ingress mainlb -n workshop
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744101217150/7b98ac41-0ea8-45c8-9142-14cc5893bda2.png align="center")

Role doesnt have permissions, so we need to provide our role with permissions

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744101306163/45ab9cd1-81e3-4fff-9a61-4b53026bff63.png align="center")

Provide Full Acces of Load Balancing so it can perform operations easily without any interruption

---

Now run “**kubectl delete -f .” to delete everything so that we can recreate**

Then hit with “kubectl apply -f .”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744101532919/c3b968c0-c152-4163-95be-4cb34668be99.png align="center")

### And now address is showing in Ingress 🎉🎉🎉

---

## Now taking this address to Namecheap (where i purchased my domain) and creating a subdomain naming challenge

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744102264144/23caa814-8782-4d1c-a272-96990cac443e.png align="center")

We have linked the load balancer URL with the challenge. cctlds.online

* This load balancer allows public access to ingress and this load balancer acts as an ingress controller
    

The ingress then routes the traffic to various services inside cluster

### So have attached url of load balancer with our domain name

## So let's access our website to see if it is running or not

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744102495375/839b7276-38ad-4000-89dc-a9add4654d7c.png align="center")

And HURRAYY !! 🎉🎉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744103189296/2730b856-d7ce-4fd1-b425-942ad3b9efe5.png align="center")

Backend is running fine too

---

# Final Thoughts:

## Deploying a real-world 3-tier MERN application on AWS EKS with a custom domain was a huge milestone for me. This project taught me how different DevOps tools and cloud services work together—from Kubernetes manifests to Ingress rules and DNS configuration.

If you're learning DevOps, cloud, or Kubernetes, this project is a great way to bring everything together in a practical scenario.

I’ll continue building on this by adding monitoring with Prometheus + Grafana, CI/CD with GitHub Actions, and security practices like network policies and secrets management. Stay tuned! 🔐🚀