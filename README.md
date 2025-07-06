# ECS to EKS/Kubernetes On-Prem Migration – PoC

## Introduction

This is a **Proof-of-Concept (PoC)** for migrating the `hello-world` app from **AWS ECS** to **Kubernetes** (AWS EKS or on-premises Minikube).  
This repo is a **reference** for architecture and migration patterns—**not a turnkey solution**.  
You’ll need to adapt the ideas and workflows here to your own environment.

---

## ECS vs EKS/Minikube – Which and Why

**ECS**  
- Managed by AWS, easy to start, but only works on AWS.
- Great for simple scaling with minimal cluster management.
- Not portable; you’re AWS-locked.

**EKS/Kubernetes (on AWS or Minikube/on-prem)**  
- The industry standard for container apps; runs on AWS or any hardware.
- Makes it easy to move between cloud and on-premises.
- Lets you use the full Kubernetes ecosystem: ArgoCD, Helm, Prometheus, etc.

|        | **ECS**                   | **EKS/K8s/Minikube**          |
|--------|---------------------------|-------------------------------|
| **Where it runs**  | AWS only                  | AWS or any server/on-prem     |
| **Flexibility**    | Limited (ECS features)    | Full K8s features             |
| **Portability**    | AWS locked-in             | Easy to move (cloud ↔ on-prem)|
| **Community**      | AWS-focused               | Huge open-source ecosystem    |
| **Migration**      | Must rewrite to leave AWS | Same setup anywhere           |

**Summary:**  
- **ECS** is simple for AWS but not portable.
- **EKS/K8s** is future-proof and can run anywhere.

---

## Why Helm?

- **Helm** is the package manager for Kubernetes.
- It makes deploying, upgrading, and rolling back apps easy.
- Keeps your deployment config in one place.

## Why Argo CD?

- **Argo CD** brings GitOps to Kubernetes.
- Watches your Git repo; applies changes automatically.
- Gives you audit, rollback, and true “Git as the source of truth”.

## Why Terraform for EKS?

- **Terraform** defines your cloud infrastructure as code (including EKS).
- Ensures repeatable, reviewable cluster setup.
- Destroy/recreate environments with confidence.

---

## Simple Migration Plan

1. **Deploy on ECS** (current).
2. **Deploy same app on EKS or Minikube** using Helm and Argo CD.
3. **Test the K8s deployment** with [k6](https://k6.io/) (use test DBs, not production!).
4. **Shift traffic gradually:** Use Route53 (or other DNS) to send 5% to K8s, 95% to ECS. Monitor everything.
5. **Increase K8s traffic** as confidence grows (30%, 50%, then 100%).
6. **Turn off ECS** after all traffic is stable on K8s.

---

## What's in this PoC

- **CI/CD Pipelines:**  
  Example GitHub Actions for build, deploy-to-ECS, and Helm chart updates (see `.github/workflows/`).
- **Sample Diagrams:**  
  Architecture diagrams are auto-generated from [`docs/structure.dsl`](docs/structure.dsl)  
  (PNG/Mermaid in [`docs/diagrams/`](docs/diagrams/)).
- **Helm chart:**  
  Example app deployment in [`helm/hello-world/`](helm/hello-world/).

---

## What’s Not Included (For Production)

- EKS cluster creation (should use Terraform)
- Logging/monitoring/alerting (add before go-live)
- Secrets encryption, RBAC, autoscaling, pod/network policies, DR/backup
- Copy-paste deployment for production

---

## How to Use This Reference

- Review and adapt the YAMLs in `.github/workflows/`
- Prepare your own AWS/K8s resources (not included)
- Use the Helm chart as a template
- Run k6 load testing
- Gradually migrate using dual-deploy and traffic-splitting

---

## Key Takeaways

- **ECS:** Simple on AWS, not portable.
- **EKS/K8s:** Portable, extensible, modern DevOps stack.
- **Helm:** Easy app deployment/rollback.
- **Argo CD:** GitOps, audit, rollback, “source of truth”.
- **Terraform:** Best-practice, reproducible cloud setup.

---

_This repo is for demo/reference only.  
Use the ideas here to guide your own migration, not for copy-paste deployment._

