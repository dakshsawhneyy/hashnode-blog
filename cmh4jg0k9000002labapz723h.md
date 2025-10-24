---
title: "Architecting for Hyperscale: The Journey from 1 to 1 Million+ Users"
datePublished: Fri Oct 24 2025 07:37:14 GMT+0000 (Coordinated Universal Time)
cuid: cmh4jg0k9000002labapz723h
slug: architecting-for-hyperscale-the-journey-from-1-to-1-million-users
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1761291337412/72795f31-74a6-430c-b2db-02eed62ccb86.png
tags: microservices, ec2, aws, cloudfront, kubernetes, scalability, infrastructure, terraform, database-replication, observability, iac, argocd, million-users, global-routing, hyperscale

---

**How this project was born:** I was reading a system design article on how to build a platform that can handle 1 Million+ users on AWS. I went through the article, but it was just a direct answer. Bottlenecks were not specified, and neither were the problems faced by infrastructure below its specifications. It was just a god-level infrastructure, which was an ideal answer to the problem.

> I thought, "Why not build the platform iteratively?" I'd start from 1 user and, while understanding the bottlenecks and problems faced by each phase, I would add resources to the infrastructure. This would be better for cost management and for understanding the *real* need for each new component.

This project is divided into five phases, each one handling a different user load.

**The Architecture:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761290944153/b3dd75f7-d438-4499-b8ec-6245356d1c0e.png align="left")

***Note:*** *The entire infrastructure for all phases was provisioned using Terraform, moving from a single file to a fully reusable, multi-region module setup.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761290956962/393ad323-bdd8-4514-9730-693a3b41506e.png align="center")

---

### Phase 0: The MVP (1 to 100 Users)

We just have one EC2 instance handling the load. The PostgreSQL database runs as a container on the same instance. Docker Compose is used to manage the containers, and a [`user-data.sh`](http://user-data.sh) script automatically configures the instance on boot.

**The Bottleneck:**

* **Single Point of Failure:** If this one instance goes down, the entire application is inaccessible.
    
* **No Scalability:** The single instance can't handle a sudden load spike.
    
* **Resource Contention:** The application and the database are fighting for the same CPU and RAM.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761291014376/fe8ae4ef-3123-4460-a0ff-b33fd34c0541.png align="center")
    

---

### Phase 1: The Growth Phase (100 to 10,000 Users)

For handling the single instance's single point of failure, we introduced multiple EC2 instances. To manage them, we added an Auto Scaling Group. To distribute the load, we added an Application Load Balancer. We also solved the resource contention by moving the database to a managed **AWS RDS** instance with Multi-AZ replication.

**The Bottleneck:**

* **Wastage of Resources:** The application was still a monolith. If Service A got 80% of the traffic and Service B only got 20%, the Auto Scaling Group would scale up the entire stack—launching new instances of both Service A and B. This meant we were paying for compute power for Service B that we didn't even need. This is the biggest disadvantage of monolithic scaling.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761291028496/f283fffd-4615-441d-a2f9-8660ef4901dd.png align="center")
    

---

### Phase 2: The Microservices Era (10,000 to 100,000 Users)

We solved high availability, but the application was still inefficient to scale and risky to deploy. This was the time to re-platform to a true microservices architecture.

**What I Built:** I provisioned a production-grade **Amazon EKS (Kubernetes)** cluster using Terraform. I deployed Service A and Service B as independent Kubernetes `Deployments` and exposed them via an **NGINX Ingress Controller**. Finally, I installed **Argo CD** and configured a full GitOps pipeline for zero-touch, automated deployments.

**The New Bottleneck:** We successfully solved independent scaling, but two new, massive problems emerged.

1. **Database Bottleneck:** With 100,000 users, the single RDS instance was overwhelmed with read requests.
    
2. **Global Latency:** The application was fast for users in India (`ap-south-1`), but users in North America or Europe were experiencing high latency.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761291048457/1bfc3807-126e-4e19-88a7-86475ef1dc67.png align="center")
    

---

### Phase 3: The Performance Tune (100,000 to 500,000 Users)

The application was now reliable but slow. The next phase was purely focused on speed and performance.

**What I Built:** I introduced two new layers to the architecture:

1. **A Caching Layer:** I provisioned an **AWS ElastiCache (Redis) cluster** and modified the application code to implement a "cache-aside" pattern. This drastically reduced the read load on the RDS database.
    
2. **A Content Delivery Network (CDN):** I placed the entire application behind an **AWS CloudFront** distribution. This cached static assets and API responses at edge locations around the world, making the application feel instant for our global users.
    

**The New Bottleneck:** The system was now incredibly fast and scalable, but it had one final, fatal flaw. The entire company was still running out of a single AWS region. If `ap-south-1` had a major outage, our entire global application would be offline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761291058995/2312723e-d8e6-4cf8-acb6-d2df70fcb9ad.png align="center")

---

### Phase 4: The Global Scale (500,000 to 1 Million+ Users)

This was the final phase: achieving true global fault tolerance and disaster recovery.

**What I Built:** I refactored the entire infrastructure into a reusable **Terraform module** (a "regional stamp").

1. **Multi-Region Deployment:** I deployed a complete, independent replica of the entire stack (EKS, RDS, ElastiCache) to a second AWS region (`us-east-1`).
    
2. **Global Routing:** I used **AWS Route 53** with **latency-based routing** to automatically direct users to the geographically closest and fastest region. This also provides automatic regional failover.
    
3. **Database Replication:** I configured the primary RDS database in Mumbai to **asynchronously replicate** to a read-replica in the `us-east-1` region, providing fast, local reads for North American users.
    

**The Result:** A globally distributed, fault-tolerant, and high-performance platform, architected to handle millions of users and survive a complete regional outage.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761291067323/e249ab41-ae05-49e9-8dd7-f482b49010f0.png align="center")

---

### Conclusion: What I Learned

This project was a journey through the real-world evolution of a high-growth startup. I didn't just build a single system; I built five, each one solving the critical bottlenecks of the last.

By scaling from 1 to 1 million users, I've learned that system design isn't about finding one "perfect" architecture. It's an iterative process of identifying the next bottleneck, applying the right SRE principle to solve it, and preparing for the new challenges that solution will create. This journey - not the final diagram is the real work of a Site Reliability Engineer.