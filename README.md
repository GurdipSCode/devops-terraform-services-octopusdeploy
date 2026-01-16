# 🐙 Octopus Deploy Terraform Service

[![Build status](https://badge.buildkite.com/your-org/your-pipeline.svg?branch=main)](https://buildkite.com/your-org/your-pipeline)
[![OpenTofu](https://img.shields.io/badge/OpenTofu-%3E%3D1.6.0-blue?logo=opentofu)](https://opentofu.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?logo=renovatebot)](https://renovatebot.com/)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

Infrastructure as Code for Octopus Deploy resources using OpenTofu/Terraform.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Repository Structure](#-repository-structure)
- [Quick Start](#-quick-start)
- [Environments](#-environments)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Configuration](#-configuration)
- [Contributing](#-contributing)

---

## 🎯 Overview

This repository manages Octopus Deploy infrastructure including:

| Resource | Description |
|----------|-------------|
| 🌍 Environments | Dev, Staging, Production environments |
| 📦 Projects | Deployment projects and process steps |
| 🔄 Lifecycles | Promotion rules and phases |
| 👥 Teams | Team permissions and scoping |
| 📚 Library Variable Sets | Shared variables across projects |
| 🎯 Tenants | Multi-tenant deployment targets |
| 🖥️ Targets | Deployment targets and workers |
| 📜 Runbooks | Operational runbooks |

---

## ✅ Prerequisites

| Tool | Version | Description |
|------|---------|-------------|
| [OpenTofu](https://opentofu.org/) | >= 1.6.0 | Infrastructure provisioning |
| [TFLint](https://github.com/terraform-linters/tflint) | >= 0.50.0 | Terraform linter |
| [Vault](https://www.vaultproject.io/) | >= 1.15.0 | Secrets management |

### Required Access

- 🔑 Octopus Deploy API key with admin permissions
- 🔐 Vault access for secrets retrieval
- 🏗️ Buildkite agent access

---

## 📁 Repository Structure

```
.
├── 📂 .buildkite/
│   ├── pipeline.yml              # Main CI/CD pipeline
│   └── scripts/                  # PowerShell automation scripts
│       ├── setup/                # Vault auth, tool installation
│       ├── validation/           # Format, validate, lint
│       ├── security/             # tfsec, checkov, trivy
│       ├── documentation/        # terraform-docs
│       ├── versioning/           # git-cliff, semantic versioning
│       ├── terraform/            # Init, plan, apply
│       └── post-deploy/          # Smoke tests, notifications
│
├── 📂 environments/
│   ├── dev/                      # Development environment
│   │   └── terraform.tfvars.example
│   ├── staging/                  # Staging environment
│   │   └── terraform.tfvars.example
│   └── prod/                     # Production environment
│       └── terraform.tfvars.example
│
├── 📂 modules/                   # Local reusable modules
│   ├── octopus-project/
│   ├── octopus-environment/
│   └── octopus-tenant/
│
├── 📄 main.tf                    # Root module
├── 📄 variables.tf               # Input variables
├── 📄 outputs.tf                 # Output values
├── 📄 providers.tf               # Provider configuration
├── 📄 versions.tf                # Version constraints
├── 📄 environments.yml           # CI/CD environment config
├── 📄 cliff.toml                 # Changelog configuration
├── 📄 renovate.json              # Dependency updates
└── 📄 README.md                  # This file
```

---

## 🚀 Quick Start

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/your-org/octopus-deploy-terraform.git
cd octopus-deploy-terraform
```

### 2️⃣ Set Up Environment Variables

```bash
export OCTOPUS_URL="https://your-octopus.example.com"
export OCTOPUS_API_KEY="API-XXXXXXXXXX"
```

### 3️⃣ Initialize OpenTofu

```bash
tofu init
```

### 4️⃣ Plan Changes

```bash
tofu plan -var-file=environments/dev/terraform.tfvars
```

### 5️⃣ Apply Changes

```bash
tofu apply -var-file=environments/dev/terraform.tfvars
```

---

## 🌍 Environments

| Environment | Auto Apply | Approval Required | Branch |
|-------------|------------|-------------------|--------|
| 🟢 Dev | ✅ Yes | ❌ No | All |
| 🟡 Staging | ❌ No | ✅ Yes | `main`, `release/*` |
| 🔴 Prod | ❌ No | ✅ Yes (platform-team) | `main` |

### Deployment Flow

```
┌─────────┐     ┌───────────┐     ┌──────────┐
│   Dev   │ ──► │  Staging  │ ──► │   Prod   │
└─────────┘     └───────────┘     └──────────┘
 auto-apply       approval         approval
                                  (platform-team)
```

---

## 🔄 CI/CD Pipeline

### Pipeline Phases

| Phase | Tools | Description |
|-------|-------|-------------|
| 1️⃣ Setup | Vault | Authenticate and install tools |
| 2️⃣ Validation | OpenTofu, TFLint | Format check, validate, lint |
| 3️⃣ Security | tfsec, checkov, trivy | Security scanning (parallel) |
| 4️⃣ Documentation | terraform-docs | Generate and check docs |
| 5️⃣ Versioning | git-cliff | Changelog and version calculation |
| 6️⃣ AI Review | fabric | AI-powered code review |
| 7️⃣ Deploy | OpenTofu | Plan and apply per environment |

### Triggering Deployments

| Action | Trigger |
|--------|---------|
| Feature branch push | Plan Dev only |
| PR to main | Plan Dev + Staging |
| Merge to main | Plan + Apply all environments |
| Manual | Buildkite UI approval |

---

## ⚙️ Configuration

### Provider Configuration

```hcl
# providers.tf
terraform {
  required_providers {
    octopusdeploy = {
      source  = "OctopusDeployLabs/octopusdeploy"
      version = "~> 0.21"
    }
  }
}

provider "octopusdeploy" {
  address = var.octopus_url
  api_key = var.octopus_api_key
  space   = var.octopus_space
}
```

### Backend Configuration

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "your-tfstate-bucket"
    key            = "octopus-deploy/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `octopus_url` | Octopus Deploy server URL | ✅ |
| `octopus_api_key` | API key for authentication | ✅ |
| `octopus_space` | Space name or ID | ✅ |
| `environment` | Target environment (dev/staging/prod) | ✅ |

---

## 🛡️ Security

### Secrets Management

All secrets are stored in HashiCorp Vault:

| Secret | Vault Path |
|--------|------------|
| Octopus API Key | `secret/octopus/api-key` |
| AWS Credentials | `secret/aws/terraform` |

### Security Scanning

| Tool | Purpose |
|------|---------|
| 🔒 tfsec | Terraform security scanner |
| ✅ checkov | Policy-as-code scanner |
| 🛡️ trivy | Misconfiguration scanner |

---

## 📝 Contributing

### Branch Naming

```
feature/OD-123-add-new-project
fix/OD-456-fix-lifecycle
chore/OD-789-update-providers
```

### Commit Messages

Follow [Conventional Commits](https://conventionalcommits.org):

```
feat(projects): add new deployment project
fix(lifecycles): correct phase ordering
docs(readme): update quick start guide
chore(deps): update octopusdeploy provider
```

### Pull Request Process

1. 🔀 Create feature branch from `main`
2. ✏️ Make changes and commit
3. 🧪 Ensure CI passes
4. 📝 Update documentation if needed
5. 🔍 Request review
6. ✅ Merge after approval

---

## 📚 Resources

| Resource | Link |
|----------|------|
| 🐙 Octopus Deploy Docs | [docs.octopus.com](https://octopus.com/docs) |
| 📦 Terraform Provider | [registry.terraform.io](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy) |
| 🌐 OpenTofu Docs | [opentofu.org/docs](https://opentofu.org/docs) |
| 🏗️ Buildkite Docs | [buildkite.com/docs](https://buildkite.com/docs) |

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built with ❤️ by the Platform Team</sub>
</p>
