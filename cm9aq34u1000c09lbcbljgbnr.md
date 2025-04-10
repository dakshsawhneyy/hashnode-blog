---
title: "DevSecOps CICD Project - SonarQube + OWASP + Trivy + Docker + Jenkins"
datePublished: Thu Apr 10 2025 02:12:09 GMT+0000 (Coordinated Universal Time)
cuid: cm9aq34u1000c09lbcbljgbnr
slug: devsecops-cicd-project-sonarqube-owasp-trivy-docker-jenkins
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744251192792/74c28d90-24d6-4146-a639-6d600897e6f6.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1744251098939/45207002-c04b-408b-be50-ccac8c8afc0f.png

---

In today’s fast-moving DevOps world, security can't be an afterthought—it needs to be baked into the pipeline. That’s where **DevSecOps** comes in.

In this project, I built a **CI/CD pipeline** that not only builds and deploys a Dockerized application but also performs **security checks at every stage** using

* **SonarQube** for code quality and static analysis
    
* **OWASP Dependency-Check** for vulnerable libraries
    
* **Trivy** for container image scanning
    
* **Docker** for containerization
    
* **Jenkins** for orchestrating the CI/CD pipeline
    

This blog walks through how I set up this end-to-end pipeline, integrated security tools into it, and followed DevSecOps principles—**shifting security left.**

---

## Procedure:

* Make a t2.medium EC2 Instance
    
* Install Docker
    
* Install Jenkins
    

---

### Let’s start with DevSecOps Practices

## Setting Up SonarQube

```bash
docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
```

Edit inbound rules and add 9000

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744191505966/4ddd2cab-ed00-4364-a87e-5e8ba25dfbf7.png align="center")

Username is admin and Password is admin

---

## Trivy Installation

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update -y
sudo apt-get install trivy -y
```

```bash
trivy --version
```

### To scan images

```bash
trivy image imagename
```

---

## OWASP Installation

Installing some plugins in jenkins

* **Go to Jenkins Master and click on <mark>Manage Jenkins --&gt; Plugins --&gt; Available plugins</mark> Install the below plugins:**
    
    * **OWASP**
        
    * **SonarQube Scanner**
        
    * **Docker**
        
    * **Pipeline: Stage View**
        

---

## Integration of SonarQube with Jenkins

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744191595440/38825019-370c-45ca-8d7e-d74e1889d979.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192141462/0ea5e3bd-27f4-4218-9080-adbefb6f63b8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192240064/9d391063-3618-4fc2-b585-55064a7b804d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192352572/6032de31-d441-4284-aec0-7a7d65b9ea8f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192501024/37489fe6-b65b-4b16-8daa-277c634f4b69.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192566254/9d6fdc78-5b82-45cb-8fe1-f320a4e339a6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192622140/3609e53f-b7be-40c3-9594-00e535481b3b.png align="center")

### Both have been integrated successfully

---

## Installing SonarQube Quality Gates

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744192930489/5b4c3ee2-2c80-4959-bfe5-d6fa4f4c82ef.png align="center")

---

## Now setting up OWASP

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744193006700/aba2f550-7c28-4800-a4ba-3212c19eb2d2.png align="center")

---

## Creating Declarative Pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744193119266/942fa7d5-f958-4ba4-bbe0-dba20f7ac094.png align="center")

## Setting Up Pipeline

```bash
pipeline{
    
    agent any
    environment{
        SONAR_HOME= tool "Sonar"
    }
    
    stages{
        stage("Code Clone from GitHub"){
            steps{
                git url: "https://github.com/dakshsawhneyy/wanderlust-project.git", branch: "master"
            }
        }
        stage("SonarQube Quality Analysis"){
            steps{
                   echo "Helloo"
                }
            }
        }
        stage("test"){
            steps{
                echo "Code is incoming"
            }
        }
        stage("deploy"){
            steps{
                echo "Code is incoming"
            }
        }
    }
}
```

### Setting up SonarQube

```bash
stage("SonarQube Quality Analysis"){
            steps{
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
```

### Setting up OWASP

```bash
stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
```

### Setting Up Trivy

```bash
stage("Trivy File System Scan"){
            steps{
                echo "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
```

### Setting Up Sonar Quality Gate Scan

```bash
stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
```

## Final Code

```bash
pipeline{
    
    agent any
    environment{
        SONAR_HOME= tool "Sonar"
    }
    
    stages{
        stage("Code Clone from GitHub"){
            steps{
                git url: "https://github.com/dakshsawhneyy/wanderlust-project.git", branch: "master"
            }
        }
        stage("SonarQube Quality Analysis"){
            steps{
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan"){
            steps{
                echo "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Docker Compose"){
            steps{
                sh 'docker compose up -d'
            }
        }
    }
}
```

## And they build the pipeline and booommmm

## Project is Completed