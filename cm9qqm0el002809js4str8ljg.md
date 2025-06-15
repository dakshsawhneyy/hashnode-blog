---
title: "Deploying WearSphere: A 4-Tier E-commerce App on AWS with DevSecOps & EKS"
datePublished: Mon Apr 21 2025 07:11:08 GMT+0000 (Coordinated Universal Time)
cuid: cm9qqm0el002809js4str8ljg
slug: deploying-wearsphere-a-4-tier-e-commerce-app-on-aws-with-devsecops-and-eks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747136914173/ad055101-364c-4232-b15d-822cc1593bb7.png
tags: microservices, docker, aws, kubernetes, automation, jenkins, e-commerce, cicd, prometheus, devsecops, grafana, eks, eks-cluster, 4-tier-app

---

### **Anyone can build a MERN app.**  
But *WearSphere* isn’t just another side project — it’s a **4-tier e-commerce platform**, **fully automated** and deployed on **AWS** using **DevOps**, **DevSecOps**, and **GitOps** best practices. From secure CI/CD pipelines to a custom domain and EKS-powered scalability, every layer was built with **real-world production** in mind.

### **Introducing *WearSphere*** — a secure, scalable, and enterprise-ready e-commerce solution engineered with modern cloud-native architecture.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745184209457/2e0205a3-df94-49d1-9015-b4bcd1b7bb28.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745184215704/3eec28bf-9519-4fb6-8046-511d0e063a57.jpeg align="center")

### GitHub Link: [https://github.com/dakshsawhneyy/WearSphere-Ecommerce-MERN.git](https://github.com/dakshsawhneyy/WearSphere-Ecommerce-MERN.git)

<mark>Project Deployment Flow:</mark>

## Tech stack used in this project:

* GitHub (Code)
    
* Docker (Containerization)
    
* Jenkins (CI)
    
* OWASP (Dependency check)
    
* SonarQube (Quality)
    
* Trivy (Filesystem Scan)
    
* ArgoCD (CD)
    
* AWS EKS (Kubernetes)
    
* Helm (Monitoring using grafana and prometheus)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745326413222/7e18cfdf-192e-4cc0-b100-2d3b3e99c604.png align="center")
    

# Mistakes I Made & What I Learned

#### **Mistake: Incorrect ALB Configuration for EKS**

* **Description:** I faced difficulties with the Application Load Balancer (ALB) setup, which led to issues with routing and service discovery in AWS EKS.
    
* **Solution:** I revisited the ALB configuration, following best practices for routing and integrating it with Kubernetes services to ensure proper traffic management.
    

**Mistake: Ignoring IAM Permissions in AWS EKS**

* **Description:** I neglected to properly configure IAM roles and security policies, leading to access and permission issues when deploying on EKS.
    
* **Solution:** After researching the required IAM configurations, I ensured that all roles and security groups were set correctly, granting the necessary permissions to the services.
    

#### **Mistake: Not Properly Managing Secrets**

* **Description:** I initially mishandled sensitive credentials, which led to security risks and potential exposure of private data.
    
* **Solution:** I adopted best practices for managing secrets using AWS Secrets Manager and Kubernetes Secrets to securely store and access sensitive information.
    

#### **Mistake: Skipping Documentation of the Deployment Process**

* **Description:** I didn’t document the deployment process properly, making it hard to revisit and explain how the app was set up later.
    
* **Solution:** I documented the entire deployment process step-by-step, including specific configurations and setup instructions, for future reference and for anyone working with the project.
    

#### **Mistake: Missing Detailed Logging and Error Handling**

* **Description:** My application initially lacked proper logging and error handling, making it difficult to diagnose issues during runtime.
    
* **Solution:** I added comprehensive logging and error handling across all layers of the application, which allowed me to easily trace and resolve issues as they arose.
    

---

### Prerequisites to implement this project:

Note:

This project will be implemented in the North California region (us-west-1).

* **Create 1 master machine on AWS with 2 CPUs, 8GB of RAM (t2.large), and 29 GB of storage, and install Docker on it.**
    
* **Open the below ports in security group of master machine and also attach same security group to Jenkins worker node (We will create worker node shortly)**
    
    ![image](https://private-user-images.githubusercontent.com/121779953/361331879-4e5ecd37-fe2e-4e4b-a6ba-14c7b62715a3.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDUxNTA5OTUsIm5iZiI6MTc0NTE1MDY5NSwicGF0aCI6Ii8xMjE3Nzk5NTMvMzYxMzMxODc5LTRlNWVjZDM3LWZlMmUtNGU0Yi1hNmJhLTE0YzdiNjI3MTVhMy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNDIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDQyMFQxMjA0NTVaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT01ZDdhNDc2MDExMjQ0NDc4YjJjNGY4MzIxMTUyMjhmY2ViZDQyMzBlMDlkZDkzYmM0ODcwOTk2N2NiOTY0MWYyJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.StDnC7j2jrydyW7j7_Kq4uqCJxNBT22OSeWZX8XbYnM align="left")
    

Note

We are creating this master machine because we will configure Jenkins master, eksctl, and EKS cluster creation from here.

Install & configure Docker by using the below command; "NewGrp docker" will refresh the group config, hence there is no need to restart the EC2 machine.

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker
```

* **Install and configure Jenkins**
    

```bash
sudo apt update -y
sudo apt install fontconfig openjdk-17-jre -y

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
  
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  
sudo apt-get update -y
sudo apt-get install jenkins -y
```

* **Now, access Jenkins Master on the browser on port 8080 and configure it**.
    
* **Create EKS Cluster on AWS (Master machine)**
    
    * IAM user with **access keys and secret access keys**
        
    * AWSCLI should be c[onfigured (Setup AWSCLI](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/AWSCLI/AWSCLI.sh))
        
    
    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip
    unzip awscliv2.zip
    sudo ./aws/install
    aws configure
    ```
    
    * Install **kubectl** ([Setup kubectl](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/Kubectl/Kubectl.sh) )
        
    
    ```bash
    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin
    kubectl version --short --client
    ```
    
    * Instal[l **eksctl**](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/eksctl%20/eksctl.sh) ([Setup](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/eksctl%20/eksctl.sh) [eksctl)](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/AWSCLI/AWSCLI.sh)
        
    
    ```bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```
    
    * **C**[**reate EKS Clus**](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/Kubectl/Kubectl.sh)**ter**
        
    
    ```bash
    eksctl create cluster --name=wearsphere \
                        --region=ap-south-1 \
                        --version=1.30 \
                        --without-nodegroup
    ```
    
    * **Associate IAM OIDC Provider**
        
    
    ```bash
    eksctl utils associate-iam-oidc-provider \
      --region ap-south-1 \
      --cluster wearsphere\
      --approve
    ```
    
    * **Creat**[**e Node**](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/eksctl%20/eksctl.sh)
        
    
    ```bash
    eksctl create nodegroup --cluster=wearsphere\
                         --region=ap-south-1 \
                         --name=wearsphere\
                         --node-type=t2.large \
                         --nodes=2 \
                         --nodes-min=2 \
                         --nodes-max=2 \
                         --node-volume-size=29 \
                         --ssh-access \
                         --ssh-public-key=eks-nodegroup-key
    ```
    

## Creating Dockerfile for frontend

```bash
FROM node:18 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

#Stage 2
FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app .

EXPOSE 5173

CMD ["npm","run","dev"]
```

```bash
docker build -t wearsphere-frontend .
```

```bash
docker run -d -p 5173:5173 wearsphere-frontend:latest
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745121542255/5456602f-5bd5-44fc-8e0a-b87a88da3ac0.png align="center")

## [Docker fi](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/Kubectl/Kubectl.sh)le for Backend

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745122465431/2b685abe-53d5-4178-84e2-dbab39e3dd0e.png align="center")

Change this with your EC2 private ip

```bash
docker build -t wearsphere-backend .
```

```bash
docker run -d -p 4000:4000 wearsphere-backend:latest
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745128191764/d99c50a2-a4ff-4b81-add3-9e8d5d8eba0b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745129342795/e927f1f1-132a-485c-8d7a-da2895dacd16.png align="center")

Backend Running Fine too

## Creating Dockerfile for Admin Panel

Change l[ocalhost wit](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/eksctl%20/eksctl.sh)h your pu[blic ip address of](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/AWSCLI/AWSCLI.sh) EC2

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745132028528/afb320a3-ecd9-4e6b-9fec-5bab487891c1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745145318429/af3e51a1-a19d-4fd8-baf7-d58449969e5e.png align="center")

Admin Panel is running too on port 5174

---

## Applying docker-compose.yml

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745128553616/c250aa72-0908-45a1-9e6d-fd196d20cd00.png align="center")

inside .env of frontend, add your public ip of ec2 instead of mine

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745128871383/8a1301e9-dc5f-46e2-b72d-4a5417b2ec5a.png align="center")

```bash
docker-compose up --build -d
```

Fo[r Adding Sample Prod](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/Kubectl/Kubectl.sh)ucts

Create seed.js inside backend folder

```bash
import mongoose from "mongoose";
import dotenv from "dotenv";
import productModel from "./models/productModel.js";

dotenv.config();

const connectDB = async () => {
  await mongoose.connect(process.env.MONGODB_URI);
  console.log("MongoDB connected!");
};

const seedProducts = async () => {
  try {
    await connectDB();

    const products = [
      {
        name: "Cool Hoodie",
        description: "Comfortable and stylish",
        price: 1299,
        image: ["https://dummyimage.com/300"],
        category: "clothing",
        subCategory: "hoodie",
        sizes: ["S", "M", "L"],
        bestseller: true,
        date: Date.now()
      },
      {
        name: "Graphic T-Shirt",
        description: "Trendy streetwear tee",
        price: 699,
        image: ["https://dummyimage.com/301"],
        category: "clothing",
        subCategory: "tshirt",
        sizes: ["M", "L"],
        bestseller: false,
        date: Date.now()
      }
    ];

    await productModel.deleteMany(); // Optional: Clears existing
    await productModel.insertMany(products);
    console.log("Sample products seeded! 🎉");
    process.exit();
  } catch (err) {
    console.error(err);
    process.exit(1);
  }
};

seedProducts();
```

Then go inside the wearsphere-backend container and run seed. js

```bash
docker exec -it wearsphere-backend sh
```

```bash
ls
```

```bash
node seed.js
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745131516861/c2cea1dd-7279-445d-a865-beaa9c2f890a.png align="center")

And Sample products Starts Coming

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745131554580/da50d9cc-3efe-4d36-a8d0-c6d28ddc15e7.png align="center")

---

## Push Images to Docker Hub

```bash
docker login
```

```bash
docker tag wearsphere-frontend dakshsawhneyy/wearsphere-frontend
docker push dakshsawhneyy/wearsphere-frontend
```

```bash
docker tag wearsphere-backend dakshsawhneyy/wearsphere-backend
docker push dakshsawhneyy/wearsphere-backend
```

```bash
docker tag wearsphere-admin dakshsawhneyy/wearsphere-admin
docker push dakshsawhneyy/wearsphere-admin
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745146141375/2d597801-527c-40d3-969a-54d95148d4ec.png align="center")

---

## Setting up Jenkins

Go to Jenkins Dashboard

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154361348/c60ef73b-90fd-4900-8300-2713b6e025b1.png align="center")

Add Credentials of Docker, Gmail, SonarQube

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154425672/458de47c-0bf9-4f70-9aca-2963f870b0a3.png align="center")

Go to SonarQube

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154469928/e72e1d0b-1b96-44db-9913-e0484708f0ff.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154497999/d81fd7d5-b6a5-47ef-bda5-63dbc5c5cf0a.png align="center")

Copy this token into Jenkins credentials

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154543258/49a3fae4-442b-444b-85cb-6712a87ab8a7.png align="center")

Now go to your gmail account &gt; manage account

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154640985/fda8dceb-cfc9-4432-b7d3-7a8fbc0130e4.png align="center")

Create one app password and paste this in jenkins

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745154725077/a9f3f0fe-9366-48d1-ba27-0795f7d677af.png align="center")

## Now, installing plugins and setting up system environments

Go to manage jenkins &gt; Plugin and install

* OWASP Dependency
    
* Stage view
    
* SonarQube Scanner
    
* SonarQube Quality Gates
    

### Go to Manage Jenkins &gt; tools

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745155903152/eabc92a6-467e-40f8-9b96-01952de8c8bb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156003837/c7e4cf83-ab39-4fcc-a36d-4f54ab632c6b.png align="center")

### Now, go to manage jenkins &gt; system

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156111466/e0f62386-bca2-4e0b-8438-2c86d587acc4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156199561/ceed122f-af1e-4e52-9b8f-03e7a030c9ef.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156273798/8bacbf7f-6e6a-4841-a515-bbe304dc4fb7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156410258/491a36c2-c976-4a98-b6bc-261a5d48961b.png align="center")

### Go to SonarQube and add webhook

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156510056/da0099b6-44f6-4292-b763-9e872146f71b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745156552100/3ddfdd9e-5471-4d2d-ada1-b5f120c73ff7.png align="center")

* **Install and configure SonarQube**
    

```bash
docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
```

* **Install Trivy**
    

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update -y
sudo apt-get install trivy -y
```

## Creating a Pipeline in Jenkins

### writing groovy pipeline script - Only for CI (initially)

```bash
Library('@Shared') _
pipeline {
    agent any
    environment{
        SONAR_HOME = tool 'Sonar'
    }
    
    stages{
        stage("WorkSpace Empty"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Git Code Clone'){
            steps{
                script{
                    clone("https://github.com/dakshsawhneyy/WearSphere-Ecommerce-MERN.git","master")
                }
            }
        }
        stage("Trivy: File Scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }
        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","wearsphere","wearsphere")
                }
            }
        }
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        stage("Docker: Build Images"){
            steps{
                script{
                    dir('backend'){
                        docker_build("dakshsawhneyy","wearsphere-backend","latest")
                    }
                    dir('frontend'){
                        docker_build("dakshsawhneyy","wearsphere-frontend","latest")
                    }
                    dir('admin'){
                        docker_build("dakshsawhneyy","wearsphere-admin","latest")
                    }
                }
            }
        }
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("wearsphere-backend","latest","dakshsawhneyy") 
                    docker_push("wearsphere-frontend","latest","dakshsawhneyy")
                    docker_push("wearsphere-admin","latest","dakshsawhneyy")
                }
            }
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177806269/ab38862f-4f69-44da-85d9-b443ee4a5dd3.png align="center")

---

## **For making it publicly accessible, we need INGRESS CONTROLLER**

Our cluster is isolated, basically, so to make it accessible to outside world, we need Ingress Controller

So for installing Ingress Controller, we need to install **HELM** !!

### Install AWS Load Balancer

**COPY**

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=ap-south-1
```

### Deploy AWS Load Balancer Controller

**COPY**

```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=three-tier-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744099217092/e161d3e5-f0d2-409d-8b9a-f8043b56b981.png?auto=compress,format&format=webp align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745169741738/175a605e-0a9b-4167-962a-99b2447680a0.png align="center")

## For deploying this on your custom domain, go to your domain purchase platform (in my case, it’s NameCheap.com)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745170046252/52081ebc-9637-4d60-82cd-2b1e3fb3d199.png align="center")

go to browser and try to open http://wearsphere.[**cctlds.online**](http://cctlds.online)

## Backend pod is crashing because it is unable to connect to backend

go inside backend folder ./backend/.env

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745170728835/4dac66e8-22e9-495c-bcb8-72ce203de67d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745174317998/79a5f413-28ee-49d3-b282-41654473b547.png align="center")

Our site is deployed on our custom domain 🎉🎉🎉

---

* **Install and Configure ArgoCD**
    
    * **Create argocd namespace**
        
    
    ```bash
    kubectl create namespace argocd
    ```
    
    * **Apply Argo manifest**
        
    
    ```bash
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
    
    * **Make sure all pods are running in argocd namespace**
        
    
    ```bash
    watch kubectl get pods -n argocd
    ```
    
    * **Install argocd CLI**
        
    
    ```bash
    sudo curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
    ```
    
    * **Provide executable permission**
        
    
    ```bash
    sudo chmod +x /usr/local/bin/argocd
    ```
    
    * **Check argocd services**
        
    
    ```bash
    kubectl get svc -n argocd
    ```
    
    * **Change argocd server's service from ClusterIP to NodePort**
        
    
    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
    ```
    
    * **Confirm service is patched or not**
        
    
    ```bash
    kubectl get svc -n argocd
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745176476062/f23b61d0-cb5e-4138-a89e-ae3a47d2196f.png align="center")
    
* Go to Browser → &lt;your-ip-of-node&gt;/port
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745176775503/ca219fea-c7b4-4343-a61c-46f2e58a0c06.png align="center")
    
    * **Fetch the initial password of argocd server**
        
    
    **COPY**
    
    ```bash
        kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
    ```
    
    * **Username: admin**
        
    * **Now, go to <mark>User Info</mark> and update your argocd password**
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745176837518/f0f9c0be-dd94-4770-a010-c1db11965aaa.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745176887907/231efb86-ad21-4ed4-971c-d4c6aa58665b.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745176964192/0e763af0-e0c6-4042-b108-949b8a4663b2.png align="center")
        

## **Adding our own eks cluster to argocd for application deployment using cli**

* **Login to Argo from CLI**
    
* **COPY**
    
    ```bash
         argocd login 52.53.156.187:32738 --username admin
    ```
    
    > ***\[!Tip\] 52.53.156.187:32738 --&gt; This should be your argocd url***
    
    ![image](https://github.com/user-attachments/assets/7d05e5ca-1a16-4054-a321-b99270ca0bf9 align="left")
    
    * **Check how many clusters are available in argocd**
        
    
    **COPY**
    
    ```bash
    argocd cluster list
    ```
    
    ![image](https://github.com/user-attachments/assets/76fe7a45-e05c-422d-9652-bdaee02d630f align="left")
    
    * **Get your cluster name**
        
    
    **COPY**
    
    ```bash
    kubectl config get-contexts
    ```
    
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177116758/647d70b2-3b40-414c-ba4f-498871ff27ed.png align="center")
        
    * **Add your cluster to argocd**
        
    
    **COPY**
    
    ```bash
    argocd cluster add  dakshsawhneyy@wearsphere.ap-south-1.eksctl.io --name wearsphere
    ```
    
    > ***\[!Tip\]*** [dakshsawhneyy@wearsphere.ap-south-1.eksctl.io](mailto:dakshsawhneyy@wearsphere.ap-south-1.eksctl.io) ***\--&gt; This should be your EKS Cluster Name.***
    > 
    > ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177225473/eb6189d4-b5ec-4f39-8db2-13df4f39cd7a.png align="center")
    
    * **Once your cluster is added to argocd, go to argocd console <mark>Settings --&gt; Clusters</mark> and verify it**
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177259741/e30e59ef-5e1e-4b86-bdb0-132c1a2f30fd.png align="center")
        
    * **Now, go to <mark>Applications</mark> and click on <mark>New App</mark>**
        
    
    > ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177331542/97f8dcb4-a336-4f58-a74d-cb7e566db5f5.png align="center")
    > 
    > ***\[!Important\] Make sure to click on the <mark>Auto-Create Namespace</mark> option while creating argocd application***
    
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177411895/684c644c-fb15-4f79-92b2-2a837742796e.png align="center")
        
    * **Congratulations, your application is deployed on AWS EKS Cluster**
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177594494/d86678c2-2721-4cff-a3db-769c430d773b.png align="center")
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745177647428/936c246c-1558-46ae-ba79-fa7f96403c4f.png align="center")

Argo CD is working perfectly fine 🎉🎉🎉

---

## Updating Jenkins file with CD

```bash
@Library('Shared') _
pipeline {
    agent any
    environment{
        SONAR_HOME = tool 'Sonar'
    }
    
    stages{
        stage("WorkSpace Empty"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Git Code Clone'){
            steps{
                script{
                    clone("https://github.com/dakshsawhneyy/WearSphere-Ecommerce-MERN.git","master")
                }
            }
        }
        stage("Trivy: File Scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }
        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","wearsphere","wearsphere")
                }
            }
        }
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        stage("Docker: Build Images"){
            steps{
                script{
                    dir('backend'){
                        docker_build("dakshsawhneyy","wearsphere-backend","latest")
                    }
                    dir('frontend'){
                        docker_build("dakshsawhneyy","wearsphere-frontend","latest")
                    }
                    dir('admin'){
                        docker_build("dakshsawhneyy","wearsphere-admin","latest")
                    }
                }
            }
        }
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("wearsphere-backend","latest","dakshsawhneyy") 
                    docker_push("wearsphere-frontend","latest","dakshsawhneyy")
                    docker_push("wearsphere-admin","latest","dakshsawhneyy")
                }
            }
        }
        stage("Update Kubernetes Manifests") {
            steps {
                script {
                    k8s_manifests('latest') 
                }
            }
        }
    }
    post {
        success {
            emailext (
                to: 'dakshsawhney2@example.com',  
                subject: "SUCCESS: WearSphere Pipeline - ${currentBuild.fullDisplayName}",
                body: """
                    The Jenkins pipeline for the WearSphere project has successfully completed.
                    Build URL: ${currentBuild.absoluteUrl}
                    Build Status: SUCCESS
                """
            )
        }
        failure {
            emailext (
                to: 'dakshsawhney2@example.com',  // Replace with the recipient's email
                subject: "FAILURE: WearSphere Pipeline - ${currentBuild.fullDisplayName}",
                body: """
                    The Jenkins pipeline for the WearSphere project has failed.
                    Build URL: ${currentBuild.absoluteUrl}
                    Build Status: FAILURE
                """
            )
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745181779115/09191935-85fc-467f-952f-4ff0346d503f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745181830035/938c19b4-0c62-44ae-875e-315c8f5f4c1b.png align="center")

---

## **How to monitor EKS cluster, kubernetes components and workloads using prometheus and grafana via HELM (On Master machine)**

* **Install Helm Chart**
    

**COPY**

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

**COPY**

```bash
chmod 700 get_helm.sh
```

**COPY**

```bash
./get_helm.sh
```

* **Add Helm Stable Charts for Your Local Client**
    

**COPY**

```bash
helm repo add stable https://charts.helm.sh/stable
```

* **Add Prometheus Helm Repository**
    

**COPY**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

* **Create Prometheus Namespace**
    

**COPY**

```bash
kubectl create namespace prometheus
```

**COPY**

```bash
kubectl get ns
```

* **Install Prometheus using Helm**
    

**COPY**

```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

* **Verify prometheus installation**
    

**COPY**

```bash
kubectl get pods -n prometheus
```

* **Check the services file (svc) of the Prometheus**
    

**COPY**

```bash
kubectl get svc -n prometheus
```

* **Expose Prometheus and Grafana to the external world through Node Port**
    

> ***\[!Important\] change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.***

**COPY**

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

![image](https://github.com/user-attachments/assets/90f5dc11-23de-457d-bbcb-944da350152e align="left")

* **Verify service**
    

**COPY**

```bash
kubectl get svc -n prometheus
```

* **Now, let’s change the SVC file of the Grafana and expose it to the outer world**
    

**COPY**

```bash
kubectl edit svc stable-grafana -n prometheus
```

![image](https://github.com/user-attachments/assets/4a2afc1f-deba-48da-831e-49a63e1a8fb6 align="left")

* **Check grafana service**
    

**COPY**

```bash
kubectl get svc -n prometheus
```

* **Get a password for grafana**
    

**COPY**

```bash
kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

> ***\[!Note\] Username: admin***

* **Now, view the Dashboard in Grafana**
    
    ![image](https://github.com/user-attachments/assets/d2e7ff2f-059d-48c4-92bb-9711943819c4 align="left")
    
    ![image](https://github.com/user-attachments/assets/3d6652d0-7795-4fe9-8919-f33eac88db73 align="left")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745182664052/364e83b8-84d7-4a65-93e5-531d1ad07e70.png align="center")
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745182745058/aa8c26c5-bed3-443e-ab88-395964dd2534.png align="center")

## **Clean Up**

* **Delete eks cluster**
    

**COPY**

```bash
eksctl delete cluster --name=wearsphere --region=ap-south-1
```