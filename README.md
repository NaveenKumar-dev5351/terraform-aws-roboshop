# 🏗️ Terraform AWS Roboshop — Golden AMI Deployment Module

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

## 🎯 Project Overview

A reusable, component-driven Terraform module implementing 
the **Golden AMI Immutable Infrastructure pattern** for 
Roboshop e-commerce microservices on AWS.

A single module deploys ANY Roboshop component — frontend, 
catalogue, user, cart, payment, shipping — by simply changing 
the `component` variable. The module automatically handles 
different ports, health check paths, ALB routing, and DNS 
records per component type.

---

## 🏗️ Architecture
                Internet
                   │
              Route53 DNS
                   │
          Frontend ALB (port 80/443)
                   │
          ┌────────┴────────┐
          │                 │
     Frontend           Backend ALB
     (port 80)          (port 8080)
          │                 │
     ┌────┴────┐    ┌───────┼───────┐
     │         │    │       │       │
  Frontend  Catalogue  User  Cart  Payment
     ASG      ASG    ASG   ASG    ASG

### Golden AMI Flow
EC2 Launch (base AMI)
↓
Bootstrap Script (install & configure app)
↓
Stop Instance
↓
Create AMI (Golden AMI)
↓
Terminate Instance
↓
Launch Template (uses Golden AMI)
↓
Auto Scaling Group
↓
Target Group + ALB Listener Rule

---

## 🧠 Smart Component Logic

The module uses conditional locals to handle 
frontend vs backend differences automatically:

| Property | Frontend | Backend Services |
|---|---|---|
| Port | 80 | 8080 |
| Health Check | `/` | `/health` |
| ALB | Frontend ALB | Backend ALB |
| DNS | `dev.devops84.store` | `catalogue.backend-dev.devops84.store` |

---

## 📦 Resources Created Per Component

| Resource | Description |
|---|---|
| `aws_lb_target_group` | Target group with health checks |
| `aws_instance` | Temporary EC2 for bootstrapping |
| `terraform_data` (bootstrap) | SSH + remote-exec provisioner |
| `aws_ec2_instance_state` | Stops instance before AMI |
| `aws_ami_from_instance` | Creates Golden AMI |
| `terraform_data` (delete) | Terminates temp instance |
| `aws_launch_template` | Launch config from Golden AMI |
| `aws_autoscaling_group` | Auto scaling with rolling refresh |
| `aws_autoscaling_policy` | Target tracking at 75% CPU |
| `aws_lb_listener_rule` | ALB routing rule |

---

## 🔗 SSM Parameter Store Integration

All cross-module values fetched from SSM — 
zero hardcoding:

| SSM Parameter | Description |
|---|---|
| `/{project}/{env}/vpc_id` | VPC ID |
| `/{project}/{env}/private_subnet_ids` | Private subnet IDs |
| `/{project}/{env}/{component}_sg_id` | Component Security Group |
| `/{project}/{env}/backend_alb_listener_arn` | Backend ALB |
| `/{project}/{env}/frontend_alb_listener_arn` | Frontend ALB |

---

## 📋 Variables

| Variable | Description | Default |
|---|---|---|
| `project` | Project name | `roboshop` |
| `environment` | Environment name | `dev` |
| `component` | Component name | Required |
| `rule_priority` | ALB listener rule priority | Required |
| `zone_id` | Route53 hosted zone ID | Required |
| `zone_name` | Domain name | `devops84.store` |

---

## 🚀 How to Use

### Deploy Catalogue Service
```bash
cd catalogue
terraform init
terraform apply \
  -var="component=catalogue" \
  -var="rule_priority=100"
```

### Deploy Frontend
```bash
cd frontend
terraform apply \
  -var="component=frontend" \
  -var="rule_priority=200"
```

### Deploy All Components
```bash
# Deploy in order — databases first
for component in mongodb mysql redis rabbitmq; do
  cd $component
  terraform apply -var="component=$component" \
    -var="rule_priority=$priority" -auto-approve
  cd ..
done

# Then application services
for component in catalogue user cart shipping payment frontend; do
  cd $component
  terraform apply -var="component=$component" \
    -var="rule_priority=$priority" -auto-approve
  cd ..
done
```

---

## ⚙️ Auto Scaling Configuration

| Setting | Value | Reason |
|---|---|---|
| Min instances | 1 | Always available |
| Max instances | 10 | Cost control |
| Desired | 1 | Start lean |
| Scale trigger | 75% CPU | Before degradation |
| Health grace period | 90s | App startup time |
| Rolling refresh | 50% healthy | Zero downtime |
| Deregistration delay | 120s | Drain connections |

---

## 🔒 Security Design

- All instances in **private subnets** — no public access
- Security Groups fetched from **SSM** — no hardcoded IDs
- AMI created from bootstrapped instance — **immutable**
- Instances terminated after AMI creation — **no lingering resources**
- Component-specific Security Groups — **least privilege**

---

## 💡 Key Design Patterns

### 1. Golden AMI Pattern
Pre-bake application into AMI — faster scaling, 
consistent deployments, no runtime dependencies.

### 2. SSM Parameter Store
Cross-module communication without remote state 
data sources or hardcoded values.

### 3. Component-Driven Design
One module handles all components — reduces 
code duplication, single source of truth.

### 4. Immutable Infrastructure
Never modify running instances — replace with 
new AMI on every deployment.

---

## 🔧 Bootstrap Script

The `bootstrapp.sh` script:
1. Receives component name and environment as args
2. Installs component-specific dependencies
3. Configures the service
4. Starts and enables the service

```bash
#!/bin/bash
component=$1
environment=$2
# Install and configure component
```

---

## 📊 Interview Talking Points

> "I implemented a Golden AMI pattern using Terraform 
> where a single reusable module deploys any Roboshop 
> microservice. The module provisions a temporary EC2 
> instance, bootstraps the application via SSH provisioner, 
> creates an AMI, terminates the instance, and deploys 
> via Auto Scaling Group with rolling instance refresh 
> and target tracking auto scaling at 75% CPU. All 
> cross-module values are fetched from AWS SSM Parameter 
> Store — zero hardcoding. The same module handles both 
> frontend and backend services through conditional 
> locals — automatically routing to correct ALB, port, 
> and health check path based on the component variable."

---

## 🔗 Related Projects
- [Roboshop AWS Infrastructure](https://github.com/NaveenKumar-dev5351/roboshop-aws-infrastructure)
- [Roboshop Docker](https://github.com/NaveenKumar-dev5351/roboshop-docker)
- [Roboshop Ansible](https://github.com/NaveenKumar-dev5351/ansible-roboshop)

## 👨‍💻 Author
**Naveen Kumar Lingampelly**
DevOps Engineer | [LinkedIn](https://linkedin.com/in/naveenlingampelli) | 
[GitHub](https://github.com/NaveenKumar-dev5351)