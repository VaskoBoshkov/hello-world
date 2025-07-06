# ECS to EKS/Kubernetes On-Prem Migration – Reference Demo (PoC)

## Introduction

This repository provides a **proof-of-concept (PoC)** and reference for migrating a containerized application (`hello-world`) from **AWS ECS** to **Kubernetes** (AWS EKS or on-premises k8s/Minikube).  
It demonstrates architecture, pipelines, and a safe migration *approach*—it is **not a plug-and-play starter kit**.

:warning: **This PoC does not provision or manage infrastructure for you.  
Do not simply clone and expect an out-of-the-box deployment.  
Adapt the patterns and workflows shown here to your team’s environment.**

---

## Objectives

- Show a safe, gradual migration from ECS to Kubernetes.
- Prove zero-downtime techniques: dual-deploy, canary/cutover via Route53 or other DNS system.
- Present a reference CI/CD workflow and migration architecture.
- Share best-practices for pre-migration validation and future production hardening.

---

## Migration Approach (Summary)

1. **Dual Deployment:**  
   - Deploy to both ECS and EKS/Kubernetes (on-prem or cloud) using the same container image and application logic. 
   - WARNING!!!- While testing the k8s/minikube app don't use the real production resources like DB's, Redis etc in the k8s/minikube deployment, 
     rather work for testing on restored DB, temp Redis etc. 

2. **Validation:**  
   - Perform stress/load testing on the new environment using (`k6.io`) before migration.
   - Ensure consistent logging between ECS and K8s.
   
3. **Canary Traffic Shift:**  
   - Use Route53 or an ALB to split live traffic:  
   - Start with 95% ECS / 5% EKS, monitor, then gradually increase EKS share (70/30, 50/50, ...).

4. **Final Cutover:**  
   - Once performance and stability are proven, route 100% of traffic to EKS/K8s and decommission ECS deployment.

---

## What's in this PoC

- **CI/CD Pipeline Examples:**  
  See `.github/workflows/` for build, deploy-to-ECS, and Helm chart update workflows.
- **Sample Structurizr DSL and Diagrams:**  
  All architectural diagrams are generated automatically from [`docs/structure.dsl`](docs/structure.dsl).  
  Diagrams are output to [`docs/diagrams/`](docs/diagrams/), always up to date.
- **Migration Mermaid Diagram:**  
  ![Diagram](docs/diagrams/structurizr-pipeline.mmd)  
  (Or view the latest in the repo.)

---

## Out of Scope for This PoC

This demo **does not** cover the following (required for production):

- EKS or cluster creation (should use Terraform/IaC in production)
- Monitoring, logging aggregation, or alerting stacks
- Encryption of secrets/config (EKS: use KMS, Sealed Secrets, etc.)
- Node termination handler, cluster autoscaler, RBAC, pod/network policies
- Disaster recovery and backup
- Copy/paste deployment for other environments

**For a production rollout, all of the above must be addressed.**

---

## Pipeline & Diagram Path

- **Workflows:**  
  - `build-image.yml`: Builds/pushes image to ECR.
  - `deploy-ecs.yml`: Conditionally deploys to ECS (toggle with repo variable).
  - `update-helm-chart.yml`: Conditionally bumps Helm chart with new tag (toggle with repo variable).
- **Architecture Diagrams:**  
  - Written in Structurizr DSL ([`docs/ci-cd.dsl`](docs/structure.dsl)).
  - Rendered as PNG/Mermaid on each push—view in [`docs/diagrams/`](docs/diagrams/).
- **Diagram update workflow:**  
  See `.github/workflows/structurizr.yml`.

---
