---
title: "DevOps Mega Project"
datePublished: Wed Apr 16 2025 16:23:35 GMT+0000 (Coordinated Universal Time)
cuid: cm9k556yr000h09jvcgma32k8
slug: devops-mega-project
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744820496611/61a7375a-9d1e-49ba-a0ee-700a79970d7c.png
tags: owasp, aws, redis, github, sonarqube, jenkins, helm, prometheus, grafana, eks, argocd, trivy

---

This will be the overall flow of this project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744709113511/b85886cc-6ec1-480f-b604-c30cb78072ed.png align="center")

---

# Wanderlust - Your Ultimate Travel Blog 🌍✈️

WanderLust is a simple MERN travel blog website. ✈ This project is aimed at helping people to contribute to open source, upskill in React, and also master Git.

![Preview Image](https://github.com/krishnaacharyaa/wanderlust/assets/116620586/17ba9da6-225f-481d-87c0-5d5a010a9538 align="left")

# Wanderlust Mega Project End to End Implementation

### In this demo, we will see how to deploy an end-to-end three-tier MERN stack application on an EKS cluster.

### <mark>Project Deployment Flow:</mark>

## Tech stack used in this project:

* GitHub (Code)
    
* Docker (Containerization)
    
* Jenkins (CI)
    
* OWASP (Dependency check)
    
* SonarQube (Quality)
    
* Trivy (Filesystem Scan)
    
* ArgoCD (CD)
    
* Redis (Caching)
    
* AWS EKS (Kubernetes)
    
* Helm (Monitoring using grafana and prometheus)
    

### How the pipeline will look after deployment:

* **CI pipeline to build and push**
    
* **CD pipeline to update application version**
    
* **ArgoCD application for deployment on EKS**
    

### Prerequisites to implement this project:

> \[!Note\] This project will be implemented on North California region (us-west-1).

* **Create 1 master machine on AWS with 2 CPUs, 8GB of RAM (t2.large), and 29 GB of storage, and install Docker on it.**
    
* **Open the below ports in security group of master machine and also attach same security group to Jenkins worker node (We will create worker node shortly)**
    
* ![image](https://github.com/user-attachments/assets/4e5ecd37-fe2e-4e4b-a6ba-14c7b62715a3 align="left")
    

> \[!Note\] We are creating this master machine because we will configure Jenkins master, eksctl, EKS cluster creation from here.

Install & configure Docker by using the below command; "NewGrp docker" will refresh the group config, hence there is no need to restart the EC2 machine.

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker
```

* **Install and configure Jenkins (Master machine)**
    

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
        
    * AWSCLI should be configured ([Setup AWSCLI](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/AWSCLI/AWSCLI.sh))
        
    
    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip
    unzip awscliv2.zip
    sudo ./aws/install
    aws configure
    ```
    
    * Install **kubectl** (master machine) (setup [kubectl](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/Kubectl/Kubectl.sh) )
        
    
    ```bash
    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin
    kubectl version --short --client
    ```
    
    * Install **eksctl** (Master machine) ([Setup eksctl](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/eksctl%20/eksctl.sh))
        
    
    ```bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```
    
    * **Create EKS Cluster (Master machine)**
        
    
    ```bash
    eksctl create cluster --name=wanderlust \
                        --region=us-east-2 \
                        --version=1.30 \
                        --without-nodegroup
    ```
    
    * **Associate IAM OIDC Provider (Master machine)**
        
    
    ```bash
    eksctl utils associate-iam-oidc-provider \
      --region us-east-2 \
      --cluster wanderlust \
      --approve
    ```
    
    * **Create Node (Master machine)**
        
    
    ```bash
    eksctl create nodegroup --cluster=wanderlust \
                         --region=us-east-2 \
                         --name=wanderlust \
                         --node-type=t2.large \
                         --nodes=2 \
                         --nodes-min=2 \
                         --nodes-max=2 \
                         --node-volume-size=29 \
                         --ssh-access \
                         --ssh-public-key=eks-nodegroup-key
    ```
    

> \[!Note\] Make sure the ssh-public-key "eks-nodegroup-key is available in your aws account"

* **Setting up jenkins worker node**
    
    * Create a new EC2 instance (Jenkins Worker) with 2 CPUs, 8GB of RAM (t2.large), and 29 GB of storage and install java on it
        
    
    ```bash
    sudo apt update -y
    sudo apt install fontconfig openjdk-17-jre -y
    ```
    
    * Create an IAM role with <mark>administrator access</mark> attach it to the jenkins worker node <mark>Select Jenkins worker node EC2 instance --&gt; Actions --&gt; Security --&gt; Modify IAM role</mark>
        
    * Configure CLI ([Setup CLI](https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/AWSCLI/AWSCLI.sh))
        
    
    ```bash
    sudo su
    ```
    
    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip
    unzip awscliv2.zip
    sudo ./aws/install
    aws configure
    ```
    
* **generate ssh keys (Master machine) to setup jenkins master-slave**
    

```bash
ssh-keygen
```

![image](https://github.com/user-attachments/assets/0c8ecb74-1bc5-46f9-ad55-1e22e8092198 align="left")

* **Now move to the directory where your SSH keys are generated and copy the content of the public key and paste it into the authorized\_keys file of the Jenkins worker node.**
    
* **Now, go to the jenkins master and navigate to <mark>Manage jenkins --&gt; Nodes</mark>, and click on Add node**
    
    * **name:** Node
        
    * **type:** permanent agent
        
    * **Number of executors:** 2
        
    * Remote root directory
        
    * **Labels:** Node
        
    * **Usage:** Only build jobs with label expressions matching this node
        
    * **Launch method:** Via ssh
        
    * **Host:** &lt;public-ip-worker-jenkins&gt;
        
    * **Credentials:** <mark>Add --&gt; Kind: ssh username with private key --&gt; ID: Worker --&gt; Description: Worker --&gt; Username: root --&gt; Private key: Enter directly --&gt; Add Private key</mark>
        
    * **Host Key Verification Strategy:** Non verifying Verification Strategy
        
    * **Availability:** Keep this agent online as much as possible
        
* And your jenkins worker node is added
    
    ![image](https://github.com/user-attachments/assets/cab93696-a4e2-4501-b164-8287d7077eef align="left")
    
* **Install docker (Jenkins Worker)**
    

```bash
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker
```

* **Install and configure SonarQube (Master machine)**
    

```bash
docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
```

* **Install Trivy (Jenkins Worker)**
    

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update -y
sudo apt-get install trivy -y
```

* **Install and Configure ArgoCD (Master Machine)**
    
    * **Create argocd namespace**
        
    
    ```bash
    kubectl create namespace argocd
    ```
    
    * **Apply argocd manifest**
        
    
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
    
    * **Check the port where ArgoCD server is running and expose it on security groups of a worker node**
        
        ![image](https://github.com/user-attachments/assets/a2932e03-ebc7-42a6-9132-82638152197f align="left")
        
    * **Access it on browser, click on Advancedproceed with**
        
    
    ```bash
    <public-ip-worker>:<port>
    ```
    
    ![image](https://github.com/user-attachments/assets/29d9cdbd-5b7c-44b3-bb9b-1d091d042ce3 align="left")
    
    ![image](https://github.com/user-attachments/assets/08f4e047-e21c-4241-ba68-f9b719a4a39a align="left")
    
    ![image](https://github.com/user-attachments/assets/1ffa85c3-9055-49b4-aab0-0947b95f0dd2 align="left")
    
    * **Fetch the initial password of argocd server**
        
    
    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
    ```
    
    * **Username: admin**
        
    * **Now, go to <mark>User Info</mark> and update your argocd password**
        

## **Steps to add email notification**

* **Go to your Jenkins Master EC2 instance and allow 465 port number for SMTPS**
    
* **Now, we need to generate an application password from our gmail account to authenticate with jenkins**
    
    * **Open gmail and go to <mark>Manage your Google Account --&gt; Security</mark>**
        

> **\[!Important\] Make sure 2 step verification must be on**

![image](https://github.com/user-attachments/assets/5ab9dc9d-dcce-4f9d-9908-01095f1253cb align="left")

* **Search for <mark>App password</mark> and create a app password for jenkins**
    
    ![image](https://github.com/user-attachments/assets/701752da-7703-4685-8f06-fe1f65dd1b9c align="left")
    
    ![image](https://github.com/user-attachments/assets/adc8d8c0-8be4-4319-9042-4115abb5c6fc align="left")
    
* **Once the app password is created, go back to jenkins <mark>Manage Jenkins --&gt; Credentials</mark> to add username and password for email notification**
    
    ![image](https://github.com/user-attachments/assets/2a42ec62-87c8-43c8-a034-7be0beb8824e align="left")
    
* **Go back to <mark>Manage Jenkins --&gt; System</mark> and search for <mark>Extended E-mail Notification</mark>**
    
    ![image](https://github.com/user-attachments/assets/bac81e24-bb07-4659-a251-955966feded8 align="left")
    
* **Scroll down and search for <mark>E-mail Notification</mark> and setup email notification**
    

> **\[!Important\] Enter your gmail password which we copied recently in password field <mark>E-mail Notification --&gt; Advance</mark>**

![image](https://github.com/user-attachments/assets/14e254fc-1400-457e-b3f4-046404b66950 align="left")

## **Steps to implement the project:**

* **Go to Jenkins Master and click on <mark>Manage Jenkins --&gt; Plugins --&gt; Available plugins</mark> Install the below plugins:**
    
    * **OWASP**
        
    * **SonarQube Scanner**
        
    * **Docker**
        
    * **Pipeline: Stage View**
        
* **Configure OWASP, move to <mark>Manage Jenkins --&gt; Plugins --&gt; Available plugins</mark> (Jenkins Worker)**
    
* **After OWASP plugin is installed, Now move to <mark>Manage jenkins --&gt; Tools</mark> (Jenkins Worker)**
    
* **Login to SonarQube server and create the credentials for jenkins to integrate with SonarQube**
    
    * **Navigate to <mark>Administration --&gt; Security --&gt; Users --&gt; Token</mark>**
        
        ![image](https://github.com/user-attachments/assets/86ad8284-5da6-4048-91fe-ac20c8e4514a align="left")
        
        ![image](https://github.com/user-attachments/assets/6bc671a5-c122-45c0-b1f0-f29999bbf751 align="left")
        
        ![image](https://github.com/user-attachments/assets/e748643a-e037-4d4c-a9be-944995979c60 align="left")
        
* **Now, go to <mark>Manage Jenkins --&gt; Credentials</mark> and add SonarQube credentials:**
    
    ![image](https://github.com/user-attachments/assets/0688e105-2170-4c3f-87a3-128c1a05a0b8 align="left")
    
* **Go to <mark>Manage Jenkins --&gt; Tools</mark> and search for SonarQube Scanner installations:**
    
    ![image](https://github.com/user-attachments/assets/2fdc1e56-f78c-43d2-914a-104ec2c8ea86 align="left")
    
* **Go to <mark>Manage Jenkins --&gt; Credentials</mark> and add GitHub credentials to push updated code from the pipeline:**
    
    ![image](https://github.com/user-attachments/assets/4d0c1a47-621e-4aa2-a0b1-71927fcdaef4 align="left")
    

> **\[!Note\] While adding github credentials add Personal Access Token in the password field.**

* **Go to <mark>Manage Jenkins --&gt; System</mark> and search for SonarQube installations:**
    
    ![image](https://github.com/user-attachments/assets/ae866185-cb2b-4e83-825b-a125ec97243a align="left")
    
* **Now again, go to <mark>Manage Jenkins --&gt; System</mark> and search for Global Trusted Pipeline Libraries:&lt;/b&gt;**
    
    ![image](https://github.com/user-attachments/assets/874b2e03-49b9-4c26-9b0f-bd07ce70c0f1 align="left")
    
    ![image](https://github.com/user-attachments/assets/1ca83b43-ce85-4970-941d-9a819ce4ecfd align="left")
    
* **Login to SonarQube server, go to <mark>Administration --&gt; Webhook</mark> and click on create**
    
    ![image](https://github.com/user-attachments/assets/16527e72-6691-4fdf-a8d2-83dd27a085cb align="left")
    
    ![image](https://github.com/user-attachments/assets/a8b45948-766a-49a4-b779-91ac3ce0443c align="left")
    
* **Now, go to the GitHub repository, and under <mark>the Automations</mark> directory, update the <mark>instance-id</mark> field on both the** **<mark>updatefrontendnew.sh and</mark>** [**<mark>updatebackendnew.sh</mark>**](http://updatebackendnew.sh) **with the k8s worker's instance ID.**
    
    ![image](https://github.com/user-attachments/assets/3cb044b4-df88-4d68-bf7c-775cf78d5bf2 align="left")
    
* **Navigate to <mark>Manage Jenkins --&gt; Credentials</mark> and add credentials for Docker login to push Docker images:**
    
    ![image](https://github.com/user-attachments/assets/1a8287fc-b205-4156-8342-3f660f15e8fa align="left")
    
* **Create a <mark>Wanderlust-CI</mark> pipeline**
    
    ![image](https://github.com/user-attachments/assets/55c7b611-3c20-445f-a49c-7d779894e232 align="left")
    
* **Create one more pipeline <mark>Wanderlust-CD</mark>**
    
    ![image](https://github.com/user-attachments/assets/23f84a93-901b-45e3-b4e8-a12cbed13986 align="left")
    
    ![image](https://github.com/user-attachments/assets/ac79f7e6-c02c-4431-bb3b-5c7489a93a63 align="left")
    
    ![image](https://github.com/user-attachments/assets/46a5937f-e06e-4265-ac0f-42543576a5cd align="left")
    
* **Provide permission to docker socket so that docker build and push command do not fail (Jenkins Worker)**
    

```bash
chmod 777 /var/run/docker.sock
```

![image](https://github.com/user-attachments/assets/e231c62a-7adb-4335-b67e-480758713dbf align="left")

* **Go to Master Machine and add our own eks cluster to argocd for application deployment using cli**
    
    * **Login to Argo from CLI**
        
    
    ```bash
     argocd login 52.53.156.187:32738 --username admin
    ```
    

> **\[!Tip\] 52.53.156.187:32738 --&gt; This should be your argocd url**

![image](https://github.com/user-attachments/assets/7d05e5ca-1a16-4054-a321-b99270ca0bf9 align="left")

* **Check how many clusters are available in argocd**
    

```bash
argocd cluster list
```

![image](https://github.com/user-attachments/assets/76fe7a45-e05c-422d-9652-bdaee02d630f align="left")

* **Get your cluster name**
    

```bash
kubectl config get-contexts
```

![image](https://github.com/user-attachments/assets/4cab99aa-cef3-45f6-9150-05004c2f09f8 align="left")

* **Add your cluster to argocd**
    

```bash
argocd cluster add Wanderlust@wanderlust.us-west-1.eksctl.io --name wanderlust-eks-cluster
```

> **\[!Tip\]** [**Wanderlust@wanderlust.us-west-1.eksctl.io**](mailto:Wanderlust@wanderlust.us-west-1.eksctl.io) **\--&gt; This should be your EKS Cluster Name.**

![image](https://github.com/user-attachments/assets/0f36aafd-bab9-4ef8-ba5d-3eb56d850604 align="left")

* **Once your cluster is added to argocd, go to argocd console <mark>Settings --&gt; Clusters</mark> and verify it**
    
    ![image](https://github.com/user-attachments/assets/4490b632-19fd-4499-a341-fabf8488d13c align="left")
    
* **Go to <mark>Settings --&gt; Repositories</mark> and click on <mark>Connect repo</mark>**
    
    ![image](https://github.com/user-attachments/assets/cc8728e5-546b-4c46-bd4c-538f4cd6a63d align="left")
    
    ![image](https://github.com/user-attachments/assets/eb3646e2-db84-4439-a11a-d4168080d9cc align="left")
    
    ![image](https://github.com/user-attachments/assets/a07f8703-5ef3-4524-aaa7-39a139335eb7 align="left")
    

> **\[!Note\] Connection should be successful**

* **Now, go to <mark>Applications</mark> and click on <mark>New App</mark>**
    

![image](https://github.com/user-attachments/assets/ec2d7a51-d78f-4947-a90b-258944ad59a2 align="left")

> **\[!Important\] Make sure to click on the <mark>Auto-Create Namespace</mark> option while creating argocd application**

![image](https://github.com/user-attachments/assets/55dcd3c2-5424-4efb-9bee-1c12bbf7f158 align="left")

* **Congratulations, your application is deployed on AWS EKS Cluster**
    
    ![image](https://github.com/user-attachments/assets/bc2d9680-fe00-49f9-81bf-93c5595c20cc align="left")
    
    ![image](https://github.com/user-attachments/assets/1ea9d486-656e-40f1-804d-2651efb54cf6 align="left")
    
* **Open ports 31000 and 31100 on worker node and Access it on browser**
    

```bash
<worker-public-ip>:31000
```

![image](https://github.com/user-attachments/assets/a4b2a4b4-e1aa-4b22-ac6b-f40003d0723a align="left")

* **Email Notification**
    
    ![image](https://github.com/user-attachments/assets/0ab1ef47-f939-4618-8651-6aa9274721f4 align="left")
    

## **How to monitor EKS cluster, kubernetes components and workloads using prometheus and grafana via HELM (On Master machine)**

* **Install Helm Chart**
    

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

```bash
chmod 700 get_helm.sh
```

```bash
./get_helm.sh
```

* **Add Helm Stable Charts for Your Local Client**
    

```bash
helm repo add stable https://charts.helm.sh/stable
```

* **Add Prometheus Helm Repository**
    

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

* **Create Prometheus Namespace**
    

```bash
kubectl create namespace prometheus
```

```bash
kubectl get ns
```

* **Install Prometheus using Helm**
    

```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

* **Verify prometheus installation**
    

```bash
kubectl get pods -n prometheus
```

* **Check the services file (svc) of the Prometheus**
    

```bash
kubectl get svc -n prometheus
```

* **Expose Prometheus and Grafana to the external world through Node Port**
    

> **\[!Important\] change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.**

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

![image](https://github.com/user-attachments/assets/90f5dc11-23de-457d-bbcb-944da350152e align="left")

* **Verify service**
    

```bash
kubectl get svc -n prometheus
```

* **Now, let’s change the SVC file of the Grafana and expose it to the outer world**
    

```bash
kubectl edit svc stable-grafana -n prometheus
```

![image](https://github.com/user-attachments/assets/4a2afc1f-deba-48da-831e-49a63e1a8fb6 align="left")

* **Check grafana service**
    

```bash
kubectl get svc -n prometheus
```

* **Get a password for grafana**
    

```bash
kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

> **\[!Note\] Username: admin**

* **Now, view the Dashboard in Grafana**
    
    ![image](https://github.com/user-attachments/assets/d2e7ff2f-059d-48c4-92bb-9711943819c4 align="left")
    
    ![image](https://github.com/user-attachments/assets/3d6652d0-7795-4fe9-8919-f33eac88db73 align="left")
    
    ![image](https://github.com/user-attachments/assets/13321ee5-5d7b-4976-b409-25d3b865a42a align="left")
    
    ![image](https://github.com/user-attachments/assets/75a22e4b-ae81-4cad-9c92-21dd90d126a8 align="left")
    

## **Clean Up**

* **Delete eks cluster**
    

```bash
eksctl delete cluster --name=wanderlust --region=us-west-1
```