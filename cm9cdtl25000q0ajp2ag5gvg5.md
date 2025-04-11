---
title: "Deploying a two-tier app on AWS"
datePublished: Fri Apr 11 2025 06:04:20 GMT+0000 (Coordinated Universal Time)
cuid: cm9cdtl25000q0ajp2ag5gvg5
slug: deploying-a-two-tier-app-on-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744354057510/4d64da24-7d4f-4d29-94fa-e370a70a9367.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1744351378391/eda4f19f-66c7-4da2-8cbf-9c913cef0b2b.png
tags: docker, aws, kubernetes, helm, docker-compose, helm-chart, two-tier-application

---

In this blog, I’ll walk you through how I deployed a real-world **two-tier application** (frontend and backend) on **AWS EC2**, using foundational **DevOps tools and AWS services** like **VPC, Security Groups, EC2 instances, and IAM**.

This project gave me hands-on experience with how cloud infrastructure is designed and deployed in a scalable, secure, and modular way—just like in production environments.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744076293796/e9f4335e-6f95-4abe-8158-b883a7d57553.png align="center")

Whether you’re a student like me or a beginner DevOps enthusiast, this guide will help you understand:

✅ Designing a two-tier architecture for the cloud  
✅ Setting up secure networking using VPC and subnets  
✅ Deploying and connecting frontend & backend services  
✅ Configuring security groups, SSH access, and traffic flow  
✅ Using key AWS services like EC2 and IAM effectively

Let’s dive in and bring this app to life on the cloud. ☁️🛠️

---

## Creating Dockerfile for Python App

```bash
FROM python:3.10 AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install mysqlclient
RUN pip install -r requirements.txt

COPY . .

# Stage 2
FROM python:3.10-slim

WORKDIR /app

COPY --from=builder /app .

RUN apt-get update && apt-get install -y \
    build-essential \
    default-libmysqlclient-dev \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 5000

CMD ["python", "app.py"]
```

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744274318616/f6d774e5-d1ec-419b-b11e-81005180e352.png align="center")

## Running MySql container as without it, python won’t run

```bash
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD="root" mysql:latest
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744274504942/7726e292-ff13-4a75-b559-518215d2559b.png align="center")

---

## Now, this Python app and MySQL are unable to connect with each other.

So we need to connect them by keeping them in the same network.

Creating a network

```bash
docker network create two-tier
```

Kill both containers and provide both with network

---

### If you see in app.py

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744275725955/6be27a26-23e1-44b5-acef-6d327159dc66.png align="center")

So we need to include it in the command of building container of flask app, as it requires this

### Creating flask container

```bash
docker run -d -p 5000:5000 --network=two-tier -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_DB=myDb two-tier-app:latest
```

### Creating MySql container

```bash
docker run -d -p 3306:3306 --network=two-tier -e MYSQL_DATABASE=myDb -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_ROOT_PASSWORD=admin mysql:latest
```

### Inspect network - which apps are there in it

```bash
docker network inspect two-tier
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744277352909/36de4451-afa4-4cd3-877c-d987cc0eb727.png align="center")

### Customizing Names

kill both container again

```bash
docker run -d -p 5000:5000 --network=two-tier -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_DB=myDb --name two-tier-app two-tier-app:latest
```

```bash
docker run -d -p 3306:3306 --network=two-tier -e MYSQL_DATABASE=myDb -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_ROOT_PASSWORD=admin --name my-sql mysql:latest
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744277554766/21677d39-6aa5-470f-88a7-ddb02173403d.png align="center")

---

### Now running this command inside mySQL for creation of table

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744279299186/0c1d9b4e-f587-4d0d-a087-bdbcff6709b6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744279366129/50e228bf-2813-4367-959d-90239adba161.png align="center")

Now inside this, write this command

```bash
CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT
);
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744279429735/97aba936-37c9-42fe-878e-ab10724ee9ae.png align="center")

---

AND WOHOOO 🎉🎉🎉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744280119376/8ade8b74-0281-4ca7-9bb0-69508a4f60e5.png align="center")

App is available to use on port 5000 and data is going into the database

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744280220076/47997d83-a887-4051-a621-29906834078b.png align="center")

---

## Pushing Docker image to Docker Hub

Do Docker login and enter docker hub credentials

Then tag the image

```bash
docker tag two-tier-app:latest dakshsawhneyy/two-tier-app:latest
```

And then push the image to docker hub

```bash
docker push dakshsawhneyy/two-tier-app:latest
```

---

### Making Docker Compose for both the containers

```bash
version: '3.9'

services:
  flask:
    container_name: two-tier-app
    image: dakshsawhneyy/two-tier-app
    ports:
      - "5000:5000"
    # Add env variables from app.py
    environment:
      - MYSQL_HOST: "mysql"
      - MYSQL_USER: "root"
      - MYSQL_PASSWORD: "admin"
      - MYSQL_DB: "myDb"
    depends_on:
      - mysql

  mysql:
    container_name: mysql
    image: mysql:latest
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE: "myDb"
      - MYSQL_USER: "admin"
      - MYSQL_PASSWORD: "admin"
      - MYSQL_ROOT_PASSWORD: "admin"
```

### As we were facing problem of tables, so we need to create volume here

Add this in mysql

```bash
volumes:
      - ./message.sql:/docker-entrypoint-initdb.d/message.sql # table copy 
      - mysql-data:/var/lib/mysql    # Data doesnt gets lost from the container
```

### Final Code

```bash
version: '3.9'

services:
  flask:
    container_name: two-tier-app
    image: dakshsawhneyy/two-tier-app
    ports:
      - "5000:5000"
    # Add env variables from app.py
    environment:
      - MYSQL_HOST=mysql
      - MYSQL_USER=root
      - MYSQL_PASSWORD=admin
      - MYSQL_DB=myDb
    depends_on:
      - mysql

  mysql:
    container_name: mysql
    image: mysql:latest
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=myDb
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=admin
      - MYSQL_ROOT_PASSWORD=admin
    volumes:
      - ./message.sql:/docker-entrypoint-initdb.d/message.sql
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744282148444/15bcf651-ef26-4c45-9d4e-88abb04f5c5d.png align="center")

And both containers are running using Docker-Compose 🎉🎉🎉

---

## **Kubernetes Architecture and Cluster Setup (Kubeadm)**

Create two instances—t2.medium, i.e., one for master and one for worker node

### Go inside [`(click here) kubestarer git repository`](https://github.com/dakshsawhneyy/kubestarter.git) to see all download links

And Install by using commands

---

## Creating Manifest Files

Making Deployment for Flask App

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-deployment
  labels:
    app: flask
spec:
  replicas: 4
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
      - name: flask-app
        image: dakshsawhneyy/two-tier-app
        ports:
        - containerPort: 5000
        env:
          - name: MYSQL_HOST
            value: "mysql"    # put mysql service ip address inside it
          - name: MYSQL_USER
            value: "root"
          - name: MYSQL_PASSWORD
            value: "admin"
          - name: MYSQL_DB
            value: "myDb"
```

### If you want to scale deployment, use code

```yaml
kubectl scale deployment flask-deployment --replicas=2
```

### MySQL Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql-cnt
        image: mysql:latest
        ports:
        - containerPort: 3306
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: "admin"
          - name: MYSQL_USER
            value: "root"
          - name: MYSQL_PASSWORD
            value: "admin"
          - name: MYSQL_DATABASE
            value: "myDb"
```

### Pod is crashing. We need to create pv and pvc

Create pv and, and app will run fine

---

## Deploying two-tier-app using HELM

```yaml
# Install HELM
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Creating Nginx Pod using Helm

```yaml
helm create nginx-chart
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744337425607/6a703812-9da2-4443-8190-cd75d1a66361.png align="center")

You can change anything in values. yml and charts gets updated by themselves

---

### Let’s start deploying two-tier-app using HELM

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744337836937/7833908d-2590-4a3c-a8b0-63d96b1b43ec.png align="center")

Now modifying values

### Now pasting environment vars of mysql into templates/deployment.yml

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744338303899/bbae9349-f451-4d8a-86dc-297475532e4a.png align="center")

### Inside template. yml, change these so we don’t need to hardcode the value

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744338495305/027205dd-bc0a-4132-b98b-32bf28be9790.png align="center")

Then package Helm Chart

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744338841100/b3bab440-7658-466e-b53b-f8237241a105.png align="center")

Now install chart

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744339004010/fed73ba4-e7fb-47d7-b601-c899ca75d732.png align="center")

---

### Error Solving

We havent provided 3306 inside security group inbound rules so we need to push 3306 inside inbound rules of master instance

Now use

```yaml
helm list # verify which helm charts are present and delete the error one
helm uninstall mysql-chart
```

Also by mistake wrote image instead of env

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744339498826/b5f89f76-7fd7-4e15-bc4c-a45f1449f539.png align="center")

### Change username to admin from root

root is already built user and we say make a user root; it cannot make user of another name  
so use username as “admin.”

Then Again install it

```yaml
helm install mysql-chart ./mysql-chart
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744340047338/5c443e8e-01fb-4709-bc62-317a9243665f.png align="center")

At least it’s running 😂. Now lets fix why it’s not working

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744341549453/d04e4807-affa-4271-901c-19dd1ef7b4d4.png align="center")

And its's running🎉🎉🎉  
Just needed to comment out the liveness probe inside values. yml

---

## Creating flask-app

helm create flask-app-chart

Just like MySQL, copy env into values. yml and then extract them into templates. yml

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744342323284/1478a7fc-fea1-43b8-afd0-c682ca150c10.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744342927405/e9135d2b-fc5a-44ff-a5da-c51862fd4579.png align="center")

### Then go inside flask-app-chart/templates/service.yaml

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744343191009/09e9c742-558b-4739-9946-66afce3b2c47.png align="center")

## And now package helm and install helm

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744343420216/6479634b-7e80-4636-ae89-9fc66c098e31.png align="center")

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744343545416/031fbc88-a65f-448c-97e4-a90b9049332b.png align="center")

Now going inside mysql to create a database

```yaml
kubectl exec -it mysql-chart-7644667ccb-5vcd4 -- /bin/bash
```

And create database—naming it “myDb“ in it

And also inside mysql and database — for table

```yaml
CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT
);
```

---

## And paste worker url on browser with port 300001

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744344195159/25c21137-d30c-4697-8a50-2f24cfabe443.png align="center")

And our site is running 🎉🎉🎉

---

And to uninstall all in one command

```yaml
# just run this command
helm uninstall flask-app-chart mysql-chart
```

And all will be deleted

---

## Deploying on EKS-Cluster

Make an EC2 “t2.micro” instance

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744345223576/660b300a-4f29-477c-bb19-ce2b5eee1278.png align="center")