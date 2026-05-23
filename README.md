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