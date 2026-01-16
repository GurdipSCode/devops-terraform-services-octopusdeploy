# devops-terraform-services-octopusdeploy

![Terraform](https://img.shields.io/badge/Terraform-OpenTofu-7B42BC?logo=terraform&logoColor=white)
![Octopus Deploy](https://img.shields.io/badge/Octopus%20Deploy-Automation-2F93E0?logo=octopusdeploy&logoColor=white)
![Build](https://img.shields.io/badge/CI-TeamCity%20%7C%20Buildkite-success?logo=buildkite&logoColor=white)
![IaC](https://img.shields.io/badge/IaC-GitOps-blue?logo=git)
![Security](https://img.shields.io/badge/Security-GitGuardian%20%7C%20Snyk-red?logo=securityscorecard)

---

## 📦 Overview

This repository defines **Octopus Deploy resources using Terraform / OpenTofu**.

It is a **service repository**, meaning it:
- **Consumes shared Terraform modules**
- **Deploys Octopus resources** such as:
  - Projects
  - Lifecycles
  - Channels
  - Environments
  - Variables & Runbooks
- Is designed to be executed **via CI/CD**, not manually

---

## 🧭 Repository Purpose

✔ Declarative management of Octopus Deploy  
✔ GitOps-friendly workflows  
✔ Environment-aware deployments  
✔ Repeatable, auditable infrastructure changes  

This repo **does not publish Terraform modules**.  
Shared modules live in separate **module repositories**.

---

## 🏗️ Architecture

```text
┌────────────┐
│   CI/CD    │  (TeamCity / Buildkite)
└─────┬──────┘
      │
      ▼
┌────────────────────────┐
│ Terraform / OpenTofu   │
│  - Service config      │
│  - Environment values  │
└─────┬──────────────────┘
      │
      ▼
┌────────────────────────┐
│ Octopus Deploy         │
│  - Projects            │
│  - Lifecycles          │
│  - Channels            │
└────────────────────────┘
