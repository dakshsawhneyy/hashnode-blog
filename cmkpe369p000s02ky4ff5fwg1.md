---
title: "Multi-Tenant Jenkins -- Secure SRE Approach"
datePublished: Thu Jan 22 2026 11:49:33 GMT+0000 (Coordinated Universal Time)
cuid: cmkpe369p000s02ky4ff5fwg1
slug: multi-tenant-jenkins-secure-sre-approach
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1769082523298/9cbb2523-f171-471c-93af-741cd8c14f4b.png
tags: azure, kubernetes, devops, sre, jenkins, devsecops, multi-tenant, aks, jenkins-devops

---

For a long time, I saw *Multi-Tenant Jenkins* mentioned repeatedly in SRE and DevOps discussions and interviews.  
I read articles, watched talks, but something always felt missing.

I understood **how** people configured it — but not **what real problem it was actually solving**.

So instead of memorizing patterns, I decided to **build it, break it, and understand it from first principles**.

This blog is a summary of that journey.

---

## Why Multi-Tenant Jenkins Exists

In production, Jenkins is rarely used by a single team.

Multiple teams share:

* The same Jenkins controller
    
* The same infrastructure
    
* The same cluster
    

And this immediately creates problems:

* Random build failures
    
* Workspace conflicts
    
* One team blocking another
    
* Credential leakage risks
    
* CI instability at scale
    

Multi-tenant Jenkins is not a feature.  
It is **a survival mechanism** for shared CI platforms.

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQGgzJOxLihOJg/feedshare-shrink_800/B56ZvSBQZiGsAg-/0/1768755103574?e=1770854400&v=beta&t=Dx2_d50NCOCF5x_cFD0T2ZTB_Not8m3dEFEbGgGAIVg align="left")

---

## Phase 1: Understanding the Workspace Problem

When a Jenkins job runs, it uses a **workspace** — a directory where:

* Code is checked out
    
* Files are created
    
* Artifacts are generated
    

In the traditional setup (Jenkins on EC2 or a VM):

* Workspaces are **reused**
    
* Builds are **stateful**
    
* Multiple jobs touch the same filesystem
    

### What I Observed

I intentionally ran concurrent builds that created the same files.

Result:

* Builds failed randomly
    
* “File already exists”
    
* “Workspace busy” errors
    
* Pipelines interfering with each other
    

This wasn’t a Jenkins bug.

This was **shared mutable state under concurrency**.

---

## Phase 2: “Why Not Just Separate Workspaces?” (And Why That Fails)

My first thought was simple:

> Why not give each job a different workspace?

But inside a **single Jenkins container or VM**:

* Disk is still shared
    
* Memory is still shared
    
* Processes still compete
    

Even with separate directories:

* One job can exhaust memory
    
* One job can slow down others
    
* One bad pipeline can poison the environment
    

Workspace separation alone is **not isolation**.

---

## Phase 3: The Real Fix — Ephemeral Agents

The real solution is **not fixing workspaces**.

The real solution is **removing shared state entirely**.

I moved Jenkins to Kubernetes and changed the execution model:

* Jenkins controller runs as **stateful control plane**
    
* Every build runs in a **new ephemeral pod**
    
* Pod is destroyed after build completion
    

**One build = one pod**

This immediately solved:

* Workspace clashes
    
* File conflicts
    
* Cross-build contamination
    
* Memory leaks from previous jobs
    

This is why Jenkins in production looks *very different* from Jenkins on EC2.

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQGP3J5tugk8bg/feedshare-shrink_2048_1536/B56ZvSBQUwHAAo-/0/1768755103199?e=1770854400&v=beta&t=TMhOM3ccpPcSObfVQy5z_H3GydHOlF3lF7krXV3ymps align="left")

---

## Phase 4: Multi-Tenancy with Folders and Credentials

Isolation is not only about compute — it’s also about **security**.

In a shared Jenkins:

* One team must never access another team’s secrets
    
* Credentials must not leak across boundaries
    

I introduced:

* Folder-based tenancy (`Team-A`, `Team-B`, `Team-C`)
    
* Folder-scoped credentials
    
* Explicit trust boundaries
    

A folder in Jenkins is not just organization —  
It is a **security and governance boundary**.

---

## Phase 5: DevSecOps with Sidecar Containers

Security scanning should not be:

* Manual
    
* Centralized
    
* Post-deployment
    

Since each build already runs in a pod, I added a **sidecar container**:

* Main container → Jenkins agent
    
* Sidecar → Trivy security scanner
    

This allowed:

* Security scans **inside the build environment**
    
* Builds to fail automatically on HIGH / CRITICAL CVEs
    
* Zero residue after build (pod destroyed)
    

This is the same **sidecar pattern** used by service meshes — applied to CI.

---

## Phase 6: Chaos Experiment — The Noisy Neighbor Problem

Even with isolated pods, another problem remained.

### Experiment

1. Removed throttling
    
2. Triggered 30 builds from Team-A
    
3. Observed cluster behavior
    

### Result

* Node pressure increased
    
* Pods stayed pending
    
* Team-B builds were blocked
    

**Isolation alone does not guarantee fairness.**

---

## Phase 7: Fixing Noisy Neighbors (SRE Thinking)

To fix this, I introduced:

* Per-team build throttling
    
* Pod CPU and memory limits
    
* Controlled concurrency per tenant
    

### Result

* Team-A could not starve the cluster
    
* Team-B remained unaffected
    
* Blast radius reduced to a single team
    

This is where CI becomes **a platform**, not a tool.

![No alternative text description for this image](https://media.licdn.com/dms/image/v2/D5622AQG0x_v65gfiTg/feedshare-shrink_2048_1536/B56ZvSBQTqIYAk-/0/1768755103142?e=1770854400&v=beta&t=mFJsFiid00cs2sYRJ6Nt_CLpOZQuIYqQFGcMfSDECDY align="left")

---

### Jenkins is not outdated.  
It’s often just **used incorrectly**.

When designed properly, it becomes a powerful, production-grade CI platform.

# Facts i discovered while researching for this project

* Most Jenkins failures are **architecture problems**, not tool issues
    
* Shared state + concurrency is the root of flaky CI
    
* Ephemeral agents eliminate entire classes of failures
    
* Multi-tenancy requires **both isolation and fairness**
    
* Security must be enforced **inside the pipeline**, not around it