# ECS to EKS/Kubernetes On-Prem Migration – PoC

## Introduction

This is a **Proof-of-Concept (PoC)** for migrating the `hello-world` app from **AWS ECS** to **Kubernetes** (AWS EKS or Minikube/on-prem).  
It demonstrates architecture, pipelines, and migration patterns—**not a turnkey solution**.  
You must adapt these ideas and workflows to your real environment.

---

## What Are We Simulating?

Currently, the `hello-world` application runs in **AWS ECS** (`ecs-hello-world.alekspetkov.com`).  
This PoC shows how to **translate** that deployment into **Kubernetes**—so you can run the same app on **AWS EKS** or on-premises with Minikube.  
We simulate a real migration, supporting dual deployments and safe traffic migration.

---

## Migration Steps

1. **Provision Kubernetes Cluster:**  
   - If using EKS (AWS), provision with **Terraform** (recommended best practice).  
   - For on-premises: use **Minikube** or your own K8s cluster.  
   - Prepare basic K8s add-ons: **Argo CD** (for GitOps) and **Helm** (for app deployment).

2. **Translate the App:**  
   - Recreate the ECS deployment as a **Helm chart** (`helm/hello-world/`).  
   - Mirror any environment variables, ports, and secrets needed by the app.

3. **Set Up CI/CD:**  
   - Use provided **GitHub Actions** workflows to build and push the app image.  
   - The pipeline is **controllable via variables**—you can choose to deploy to ECS, EKS, or both by toggling repository variables.  
   - See `.github/workflows/` for YAMLs.

4. **Dual Deployment:**  
   - With variables enabled, CI/CD can deploy to **both ECS and EKS/K8s**—simultaneously.  
   - Now, you have two live endpoints:  
     - **ECS:** [ecs-hello-world.alekspetkov.com](https://ecs-hello-world.alekspetkov.com)  
     - **K8s:** [hello-world.alekspetkov.com](https://hello-world.alekspetkov.com)

---

## Live Demo & How to Trigger Deployments

- **Edit the app:** Make a change (like a visible string) in `Program.cs`.  
- **Commit & Push:**  
  The GitHub Actions pipeline will build the image and deploy based on your variable settings.  
- **Observe:**  
  - **ECS deployment:** updates at [ecs-hello-world.alekspetkov.com](https://ecs-hello-world.alekspetkov.com)  
  - **K8s deployment:** updates at [hello-world.alekspetkov.com](https://hello-world.alekspetkov.com)  
- **CI/CD pipeline** handles image tagging, Helm chart versioning, and environment-specific deployments.

---

## How Traffic Cutover Would Happen

In a real migration, **Route53** (or your DNS provider) is set up to split traffic between both deployments:

- Start with 95% to ECS, 5% to K8s (weighted DNS).  
- Gradually increase K8s share as confidence grows (monitor logs and errors!).  
- Eventually, 100% to K8s, 0% to ECS—then you can decommission ECS.

---

## CI/CD Control

- Pipelines are designed so **deployments to ECS/EKS are toggled by repo variables** (no code changes needed).  
- You can test dual-deployment, ECS-only, or K8s-only by simply changing the variable in GitHub repository settings.

---

## Real-World Migration Disclaimer

> **Weighted DNS routing only works safely if users can hit either ECS or EKS backend interchangeably without losing data or session state.**  
> For this, the following must be true:  
> - Sessions, caches, databases, and other stateful components must be **shared and accessible from both ECS and EKS** (e.g., shared Redis or database).  
> - The app must **not store critical session or game state only in memory** on individual pods or tasks.  
> - Health checks must be attached to each weighted DNS record so traffic is routed away from unhealthy targets.  
> - DNS TTL values should be low (e.g., ≤ 60 seconds) for fast traffic shifts and rollbacks.  
>  
> If these conditions are not met—such as in apps with sticky WebSockets, in-memory session state, or other stateful connection requirements—**weighted DNS alone is insufficient for zero-downtime migration**.  
> In such cases, consider using:  
> - Load balancer sticky sessions (ALB/NLB cookie affinity),  
> - Service mesh traffic splitting, or  
> - Blue/green or canary deployments with planned maintenance windows for cutover.

---

## Why Helm?

- **Helm** is the package manager for Kubernetes—makes configuration, upgrades, and rollback easy.  
- Keeps all deployment settings together and reproducible.

## Why Argo CD?

- **Argo CD** enables **GitOps**: your cluster matches what’s in Git—every change is visible, auditable, and repeatable.  
- Automatic sync, rollback, and cluster drift correction.

## Why Terraform for EKS?

- **Terraform** lets you define EKS infrastructure as code (repeatable, reviewable, and safe).  
- Provision, update, or destroy clusters confidently.

---

## ECS vs EKS/K8s Comparison

|        | **ECS**                   | **EKS/K8s/Minikube**          |
|--------|---------------------------|-------------------------------|
| **Where it runs**  | AWS only                  | AWS or on-prem/any server     |
| **Flexibility**    | Limited (ECS features)    | Full K8s features, any cloud  |
| **Portability**    | AWS locked-in             | Easy to move (cloud ↔ on-prem)|
| **Community**      | AWS-focused               | Huge open-source ecosystem    |
| **Migration**      | Must rewrite to leave AWS | Same setup anywhere           |

---

## What’s in this PoC

- **CI/CD pipelines:** GitHub Actions for building, deploying to ECS, and updating Helm for K8s.  
- **Helm chart:** in [`helm/hello-world/`](helm/hello-world/).  
- **Sample diagrams:**  
  - Structurizr DSL at [`docs/structure.dsl`](docs/structure.dsl)  
  - Generated diagrams in [`docs/diagrams/`](docs/diagrams/)  
- **App:** Minimal `hello-world` ASP.NET Core example.

---

## What’s Out of Scope (for Production)

- Automated EKS/Minikube provisioning  
- Centralized logging, monitoring, alerting  
- Secrets encryption (KMS, Sealed Secrets)  
- Node termination handler, autoscaling, advanced RBAC, pod/network policies, DR/backup

---

## Summary

- This PoC shows **how** to migrate an app from ECS to EKS/K8s/on-prem, keep both running, and control rollout.  
- All code and infrastructure is **reference only**—you’ll need to adapt for your organization.  
- Focus is on pipeline and migration approach, not on full production readiness.

---

_Questions or want to extend this PoC?  
Fork and PRs are welcome!_
