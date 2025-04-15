---
title: "Deployment of Book My Show App"
datePublished: Tue Apr 15 2025 09:03:35 GMT+0000 (Coordinated Universal Time)
cuid: cm9i9zhym000a08l53ebvg0d7
slug: deployment-of-book-my-show-app
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744707781722/47cb9230-5fb6-451d-9328-19840ba069ca.png

---

Create a t2.large EC2 Instance

Open some inbound rules for the instance

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744433947366/6c0855b9-6da5-4930-92cc-9309e7df2503.png align="center")

---

## Installation of Jenkins, SonarQube, Docker And Trivy

Jenkins

```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.13" 2024-10-15
OpenJDK Runtime Environment (build 17.0.13+11-Debian-2)
OpenJDK 64-Bit Server VM (build 17.0.13+11-Debian-2, mixed mode, sharing)

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

SonarQube

```bash
docker run -d -p 9000:9000 sonarqube:lts-community
```

Trivy

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

And then configure aws cli by using

```bash
aws configure
```

Installing Kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

Installing EKSCTL - required to make EKS Cluster

```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

---

## Creating EKS Cluster

```bash
# it will only create control plane, no nodes
eksctl create cluster --name book-my-show --region ap-south-1 --without-nodegroup
```

---

## Configuring Tools in Jenkins

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744439783605/6f0c410f-4b92-4e84-8ee4-f551d9d6c1bd.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744440071235/082e8bbd-907e-4ece-a870-d6a0d3b9da20.png align="center")

### Adding Docker-Hub Credentials as well

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744440203804/04beccfe-123e-490c-a276-a5d81e007066.png align="center")

### Creating SonarQube WebHooks

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744440295554/121d824a-71c7-4baf-8365-bbdc157204a5.png align="center")

### App is made in java, so we need to configure JDK - Manage Jenkins/Tools

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744442029610/f1d61a6f-3b53-436b-97b9-c0318d02b129.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744442076646/80d9029e-1b7a-43ba-b675-f5b1340c09ce.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744442114315/3b0b0669-b801-4f06-b0c1-19104dd840fb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744442168639/922d355d-3d56-4a7e-8ac9-f819fa13dd39.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744442221435/3b216d42-fd03-4076-bab8-7d5756849411.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744442557815/62273b10-9ef4-4457-ab8e-809f28f75849.png align="center")

---

### We also need to configure email

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744443347433/3737cffc-0de7-483a-bc05-dc16669f0490.png align="center")

Remember to remove spaces between generated app token while pasting in password

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744443443210/04a51b58-d81e-443a-997d-b2372c3bd6b1.png align="left")

### We need notification, so we need to configure email in jenkins

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744443620280/7a69e5d0-f0c4-4a66-8070-4f3cc8adb7d1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744443789706/16b66bc8-4f8b-4753-b134-28ac65bea2c8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744443981601/0b4b7daa-27b1-4a37-9bd5-5bbf513b21c1.png align="center")

---

## Installing NPM

```bash
sudo apt install npm
```

## Then we need to associate EKS cluster with OIDC provider

```bash
eksctl utils associate-iam-oidc-provider \
--region ap-south-1 \
--cluster book-my-show \
--approve
```

## Creating Node Group

```bash
eksctl create nodegroup --cluster=book-my-show \
--region=ap-south-1 \
--name=node2 \
--node-type=t3.medium \
--nodes=3 \
--nodes-min=2 \
--nodes-max=4 \
--node-volume-size=20 \
--ssh-access \
--ssh-public-key=general-key-pair \
--managed \
--asg-access \
--external-dns-access \
--full-ecr-access \
--appmesh-access \
--alb-ingress-access
```

---

## Creating Jenkins Pipeline

```bash
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/dakshsawhneyy/Book-My-Show-App.git'
                sh 'ls -la'  // Verify files after checkout
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BMS \
                    -Dsonar.projectKey=BMS 
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app
                ls -la  # Verify package.json exists
                if [ -f package.json ]; then
                    rm -rf node_modules package-lock.json  # Remove old dependencies
                    npm install  # Install fresh dependencies
                else
                    echo "Error: package.json not found in bookmyshow-app!"
                    exit 1
                fi
                '''
            }
        }
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh ''' 
                        echo "Building Docker image..."
                        docker build --no-cache -t dakshsawhneyy/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app
                        echo "Pushing Docker image to registry..."
                        docker push dakshsawhneyy/bms:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                sh ''' 
                echo "Stopping and removing old container..."
                docker stop bms || true
                docker rm bms || true

                echo "Running new container on port 3000..."
                docker run -d --restart=always --name bms -p 3000:3000 dakshsawhneyy/bms:latest

                echo "Checking running containers..."
                docker ps -a

                echo "Fetching logs..."
                sleep 5  # Give time for the app to start
                docker logs bms
                '''
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                      "Build Number: ${env.BUILD_NUMBER}<br/>" +
                      "URL: ${env.BUILD_URL}<br/>",
                to: 'dakshsawhney2@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
```

And if we see in SonarQube, we see

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744452240345/bd74b17b-aa16-4737-b95a-b2573f648de3.png align="center")

If we click on “build now,” the jenkins pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744453524382/8714cb64-d3ee-429f-8073-8a7ab0b4e9e1.png align="center")

### Till this step, we haven’t deployed it on the EKS cluster. We just have to dockerize the app

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744458394013/9c84c392-c185-41e7-bf7c-2381dec742fd.png align="center")

And our website is deployed perfectly

---

## Till now, I have only deployed it on Docker.

Will further deploy it on EKS Cluster as well

(Got exhausted by writing jenkins groovy file is silent. 🥲🥲🥲)