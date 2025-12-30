---
title: "AIOps-Driven Self Healing SRE Platform"
datePublished: Tue Dec 30 2025 10:39:50 GMT+0000 (Coordinated Universal Time)
cuid: cmjsggx89000602i28kjc1my5
slug: aiops-driven-self-healing-sre-platform
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1767091063202/43b1d6c7-d9cf-4563-bd0f-84d0c8e4d37d.jpeg
tags: cloud, aws, azure, devops, sre, kinesis, s3, supervised-learning, mlops, aks, aiops, 3-tier-app, fluentbit, normalizer, self-healing-systems

---

This is not a copy-paste project.  
This is not a “**run these commands and you’re done**” blog.

This project came from system-level thinking - breaking a complex problem into smaller, solvable components and stitching them back into a single, intelligent platform.

I wanted to explore a simple but difficult question:  
**What happens when DevOps, SRE, Cloud, and MLOps stop behaving like separate tools and start working as one system?**

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQGEI04tMW-cnw/feedshare-shrink_2048_1536/B56Zscfda2KEAw-/0/1765709570310?e=1768435200&v=beta&t=-3pdcF-eXPn_iqtDVcvN-VqaOtLdpcbRgSMFywrms1Q align="left")

---

## The Core Idea

Instead of treating DevOps, SRE, Cloud, and MLOps as separate disciplines, I wanted them to operate as **a single feedback loop**:

Observe → Learn → Decide → Act → Improve.

Not by calling an external AI API or embedding a large language model, but by designing a system that *learns from its own failures* and reacts based on prior behavior.  
The goal was not to “add AI”, but to let the platform **think operationally**.

### Why Multi-Cloud Was Intentional

This was not my first multi-cloud project, and I did not introduce multiple clouds to demonstrate Terraform abstractions or provider switching.

The decision was practical and architectural.  
Running Kubernetes clusters consumes the majority of cloud costs. As a student, maintaining EKS long-term was not viable. At the same time, I had stronger operational experience with AWS services.

So I deliberately chose:  
\- **Azure AKS** to run Kubernetes workloads  
\- **AWS** to power the entire AIOps and automation layer

This forced me to solve a real problem enterprises face: **securely integrating services across cloud boundaries**.

### Building the Observability Backbone

The foundation of the system is observability.

I deployed Fluent Bit as a DaemonSet on AKS to collect node-level and pod-level metrics and logs. These signals are streamed in real time to AWS Kinesis, where they enter a centralized processing pipeline.

Raw telemetry is noisy and inconsistent, so the first compute layer is a **normalization Lambda**. Its responsibility is deliberately simple:  
— Identify whether incoming data is a metric or a log  
— Normalize structure and timestamps  
— Persist clean records into an S3 data lake

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQHPpUMFi8YWBA/feedshare-shrink_2048_1536/B56ZscfdadKEAw-/0/1765709568997?e=1768435200&v=beta&t=Y8Ffg5tkFodeAzARc20Euaj8YEiRn9t9npFszg0t1M8 align="left")

At this stage, nothing intelligent happens. This is intentional. Intelligence without clean data is unreliable.

## Turning Chaos Into Training Data

Anomaly detection is only as good as the data it learns from.  
Instead of relying on synthetic datasets, I introduced **controlled chaos** into the system.

I intentionally generated:  
— CPU saturation  
— Memory leaks  
— Traffic floods  
— Pod restarts

For each experiment, I created **ground-truth windows** representing when the system was under stress. These windows are stored as labeled events in S3.

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQHf8FWrdaUo_w/feedshare-shrink_2048_1536/B56ZscfdbXG4A4-/0/1765709569883?e=1768435200&v=beta&t=uGJ0NmoRawzI89Nb9U1r4G9XSxY6J6Y78rHkpg8v_-w align="left")

A secondary Lambda process uses these windows to extract relevant metric slices from the normalized data and convert them into structured training datasets. In effect, the platform learns directly from how it breaks.  
This approach converts chaos engineering into a **supervised learning pipeline**.

## MLOps Phase (kept intentionally simple)

The machine learning component is deliberately minimal and explainable.

For each metric type **(CPU, memory, traffic)**, I trained an independent Isolation Forest model. These models are trained offline using the generated datasets and stored as versioned artifacts in S3.  
There is no black-box modeling, no large foundation models, and no unnecessary abstraction. The focus is operational reliability, not model sophistication.

During runtime, an inference Lambda consumes normalized metrics from the stream, loads the appropriate model, and evaluates whether behavior is anomalous. When an anomaly is detected, the system assigns severity, records the incident, and emits an event.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767089272677/330253f6-0cb7-49ad-a12e-80e11689267e.png align="center")

This is where **AIOps becomes operational**, not theoretical.

## Detection without action is just monitoring.

When an incident is detected, the inference layer publishes an event to SNS. This event triggers the **auto-healing controller**, implemented as a Lambda function.

The challenge here was not logic -- it was **secure cross-cloud execution**.

To solve this, I deployed a minimal auto-heal API inside the AKS cluster with strict RBAC permissions. AWS Lambda communicates with this API using:  
— Secrets stored in AWS Secrets Manager  
— Signed custom headers  
— Least-privilege Kubernetes service accounts

Based on the incident type, the controller performs targeted actions such as:  
— Scaling deployments  
— Restarting unstable workloads  
— Stabilizing traffic handling

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQGbmmHYYa3DRA/feedshare-shrink_2048_1536/B56ZscfdbiKEAw-/0/1765709570467?e=1768435200&v=beta&t=ls9AFegDlWpaxLRWskHuaITErOKZ5Q91W-6ZSrkRBIY align="left")

Every action is logged into DynamoDB to preserve a complete incident history.  
No manual intervention is required.

---

## Why This Is AIOps, MLOps, DevOps, and SRE

This system qualifies as:

* **DevOps** because infrastructure, pipelines, and deployments are automated end-to-end
    
* **SRE** because reliability is enforced through observability, chaos engineering, and automated remediation
    
* **MLOps** because models are trained, versioned, and used in production decision loops
    
* **AIOps** because operational decisions are driven by learned behavior, not static rules
    
* ```plaintext
    Artificial intelligence here does not mean language models.
    It means systems that reason about their own state and act accordingly.
    ```