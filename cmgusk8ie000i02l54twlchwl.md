---
title: "Multi Cloud Orchestration"
datePublished: Fri Oct 17 2025 11:54:46 GMT+0000 (Coordinated Universal Time)
cuid: cmgusk8ie000i02l54twlchwl
slug: multi-cloud-orchestration
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760764242069/ac68ae8b-e76b-4e44-b160-b72ca84ad6e0.png
tags: aws, azure, ansible, cloud-computing, devops, sre, terraform, multi-cloud, observability, iac, datadog

---

### Why Multi Cloud ?

> In an era defined by cloud computing, the debate is often framed as AWS vs. Azure. But the real strategic problem isn't picking a vendor; it's the risk of being controlled by one. Relying on a single cloud provider introduces critical risks:

* **Vendor Lock-in:** Dependent on single cloud provider and even after finding their services costly, one cannot switch to another provider.
    
* **Single Point of Failure:** Vulnerability to region-wide outages or provider-level security breaches that can take your entire business offline.
    
* **Sub-Optimal Tooling:** Being forced to use a provider's less-than-ideal service when a competitor offers a superior, more cost-effective solution.
    
* **Not able to switch:** No cloud provider is best. Each has its flaws. By using single provider, we use their flaws as well which affect the organiszation
    
* **Resilience**: Imagine you are using AWS for the deployment and infrastructure, and your application is deployed in ap-south-1. What if there's an outrage in Mumbai region. Now you will say just deploy replica of application in another region na simple as that. But there will be still a chance of security breach, global authentication failure or increased latency.
    
* **Cost and Performance**: No cloud provider is perfect in everything. AWS might have variety of services and features but still if you want to work with AI/ML services, use of GCP would be more efficient. If you use only one provider, you will be forced to use their services even if they charge more for services.
    

## The solution: Multi Cloud

Multi cloud doesn't mean using multiple cloud platforms but instead, it is a business strategy to solve the problems mentioned above

## The Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1760695089496/efe99a40-70ca-4059-9c8e-08754adfa88a.png align="left")

1. Solved the problem of "vendor lock-in": We are now aren't bounded to single cloud provider but instead we can switch from AWS to AZURE or vice-versa with just one terraform command `terraform workspace select azure` and `terraform workspace select aws` with a single configuration file. No need to write different IAC for different cloud providers. We can easily switch between any of the cloud provider's infrastructure.
    
2. Solved Resilience Problem: If any of the infrastructure gets down, Route53 will detect the change and immediately send the user's request to the infrastructure deployed on another cloud platform leading to zero down time even if one cloud provider fails as a whole.
    
3. Optimized Cost and Performance: We can easily switch between any of the cloud providers due to which we can use services as per the pricing and as per our need leading to decrease in overall cost spent on DevOps and cloud practices.
    

### How i solved it 😈

* Switching between different cloud providers with just one command: I have used the concept of terraform workspaces so that we can easily switch between providers as per our need and requirements. We can write different IAC for different providers but since workspaces helps in reusability of code and helps in maintaining consistency through the infrastructure. It also solves the problem of Configuration Drift.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1760695108458/89ec35aa-0416-42e4-9cdc-99eeb12594ed.png align="left")
    
* Using Ansible for Configuration Management: Instead of trying to ssh into all indvijual instances. I used Ansible along with CI to directly install NginX and Datadog assistant using ansible. The problem here was we needed a dynamic inventory where ips of instances should be fetches directly from Terraform after creation so i created a template and a local file in Terraform which injected the values of ip and created an inventory file. This inventory file then is used in CI to install all required packages by itself
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1760695116403/21b65647-3c98-48e3-910d-986c45b489a7.png align="left")
    
* Dynamic Index.html deployment using Nginx: For the differentiation of different cloud providers, i used concept called dynamic html files which renders content based on cloud providers. I created a dynamic indes.html.j2 file which renders content based on cloud provider handling request.
    
* Dynamic Nginx Config file: To deploy dynamic web page on different provider instances, i created dynamic nginx config file that deploys app based on cloud providers
    
* Tracing and Logging: Observability was my top most priority as i believe in "If something can't be seen, it can't be solved". So i integrated DataDog agent in each of VM and instances so it can fetch real time data, and i can actually compare them in real time.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1760695140912/7b7a0359-ef38-4e16-8f76-948d2af1a8a1.png align="center")
    

## Challenges & Key Learnings

* SSH-Key Error: I stored the private ssh key in GitHub Actions secrets and i a blank line at the end of the private key left off due to which it caused errors for Ansible to ssh into instances and in configuration. So i learnt that in hard way that there should not be even an extra blank space in private key.
    
* First Destroy then Apply: I then created new ssh key, changed tf config and like then also it was not working even when i was doing terraform apply. But then i realised it is making changes in existing instances only. For instance key to be changed, i need to first destroy the instance. So then i destroyed the infra and created again which fixed this Ansible SSH issue
    
* Copy vs Template: I created a dynamic index.html file which renders content based on the cloud provider you using. I didn’t thought of creating that but instead it was a solution to a problem i faced. In html file, terraform needs to inject the provider so Nginx can deploy that specific webpage for user to see which provider they using. I was using the copy command in Ansible Playbook and i realized this is simple command. It is not powerful enough to inject variables. So then i thought to create a template. Inject variables into it and then copy to /var/www/daksh/index.html so nginx can deploy that.
    
* Nginx Duplicate Error: In Ansible configuration playbook, i created path /var/www/html/daksh and copied the index.html in here. Then when Ansible started the configuration, it thrown error of duplicate default server. I surfed the internet and was not able to find the solution for this problem. Then i randomly saw a stackoverflow article and he said to delete the default nginx server. So i wrote an additional step to remove the default nginx server page always and then try to deploy my custom server page.