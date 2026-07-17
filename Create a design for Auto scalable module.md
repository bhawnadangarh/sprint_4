
# AWS Auto-Scalable Compute Infrastructure - Terraform Module

<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/876d2897-8dcd-4265-8134-df2826d364b6" />


## Document Details

| Author         | Created    | Version | Last Updated By | Last Edited On | L0 Reviewer                       | L1 Reviewer | L2 Reviewer    |
| -------------- | ---------- | ------- | --------------- | -------------- | --------------------------------- | ----------- | -------------- |
| Bhawna Dangarh | 2026-07-17 | 1.0     | Bhawna Dangarh  | 2026-07-18    | Sharvari Khamkar / Tina Bhatnagar | Aman Raj    | Abhishek Dubey |

# Table of Contents

[Document Details](#document-details)

1. [Objective](#1-objective)
2. [Scope](#2-scope)
3. [Terraform Resources Used](#3-terraform-resources-used)
4. [Terraform Data Sources Used](#4-terraform-data-sources-used)
5. [Module Structure](#5-module-structure)
6. [Resource Interdependency Flow](#6-resource-interdependency-flow)
7. [File Description](#7-file-description)
8. [Input Variables](#8-input-variables)
9. [Instance Design](#9-instance-design)
10. [Security Group Design](#10-security-group-design)
11. [Security Group Best Practices](#11-security-group-best-practices)
12. [Scaling Policy Design](#12-scaling-policy-design)
    1. [Scale Out Policy](#121-scale-out-policy)
    2. [Scale In Policy](#122-scale-in-policy)
13. [Best Practices Followed](#13-best-practices-followed)
14. [References](#14-references)
15. [Contact Information](#15-contact-information)

## 1. Objective

This Terraform module provisions a reusable AWS Auto Scaling infrastructure. It creates a Launch Template, Auto Scaling Group (ASG), CloudWatch-based scaling policies, and integrates with existing Security Groups and ALB Target Groups. The module is designed for reuse across multiple applications and environments.

---

## 2. Scope

- Create Launch Template
- Create Auto Scaling Group
- Configure CPU-based auto scaling
- Support existing AWS resources using data sources
- Attach ASG to ALB Target Groups
- Export outputs for reuse

---

## 3. Terraform Resources Used

| Resource | Purpose |
|----------|---------|
| aws_launch_template | Defines EC2 configuration |
| aws_autoscaling_group | Creates and manages scalable EC2 instances |
| aws_autoscaling_policy | Configures scaling actions |
| aws_cloudwatch_metric_alarm | Triggers scaling based on CPU |
| aws_security_group | Creates Security Group (optional) |
| aws_lb_target_group_attachment | Attaches instances to Target Group |
| aws_iam_instance_profile | Associates IAM role with EC2 |

---

## 4. Terraform Data Sources Used

This module uses Terraform data sources to retrieve existing AWS resources such as VPCs, subnets, security groups, IAM instance profiles, AMIs, target groups, and key pairs instead of creating them again.

| Data Source | Purpose |
|------------|---------|
| data.aws_vpc | Existing VPC |
| data.aws_subnets | Existing subnets |
| data.aws_security_group | Existing Security Group |
| data.aws_lb_target_group | Existing Target Group |
| data.aws_iam_instance_profile | Existing IAM Profile |
| data.aws_ami | Existing AMI |
| data.aws_key_pair | Existing Key Pair |

---

## 5. Module Structure

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/9af62e42-19b9-4bf9-804d-4bdb14015344" />

    
---

# 6. Resource Interdependency Flow

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/a3aa5e2b-2588-4b75-8a76-cc18261565e1" />


---

## 7. File Description

| File | Purpose |
|------|---------|
| provider.tf | AWS Provider configuration |
| versions.tf | Terraform and provider versions |
| variables.tf | Input variables |
| data.tf | Existing AWS resources |
| main.tf | Infrastructure resources |
| outputs.tf | Output values |
| terraform.tfvars | Environment-specific values |
| README.md | Module documentation |
| examples/basic | Example usage |

---

## 8. Input Variables

The following are the primary input variables required to configure the Auto-Scalable Compute Infrastructure module.

| Variable | Type | Description |
|----------|------|-------------|
| `name` | `string` | Resource name prefix |
| `environment` | `string` | Deployment environment (e.g., dev, qa, prod) |
| `vpc_id` | `string` | Existing VPC ID |
| `subnet_ids` | `list(string)` | Subnet IDs where the Auto Scaling Group will be deployed |
| `ami_id` | `string` | EC2 AMI ID used by the Launch Template |
| `instance_type` | `string` | EC2 instance type |
| `iam_instance_profile_name` | `string` | IAM Instance Profile attached to EC2 instances |
| `min_size` | `number` | Minimum number of instances in the Auto Scaling Group |
| `max_size` | `number` | Maximum number of instances in the Auto Scaling Group |
| `desired_capacity` | `number` | Desired number of running instances |
| `security_group_ids` | `list(string)` | Security Group IDs associated with EC2 instances |
| `target_group_arns` | `list(string)` | ALB Target Group ARNs for traffic routing |
| `scale_up_cpu_threshold` | `number` | CPU utilization threshold to trigger Scale Out |
| `scale_down_cpu_threshold` | `number` | CPU utilization threshold to trigger Scale In |
| `tags` | `map(string)` | Additional tags applied to AWS resources |

## 9. Instance Design

Choose the instance type based on your application's CPU and memory requirements. Lightweight services can use smaller instances, while Java or high-traffic applications may require larger instance types.

---

## 10. Security Group Design

The module supports two approaches:

- Use an existing Security Group.
- Create a new Security Group through the module.

Ingress and egress rules are configurable using input variables.

---

## 11. Security Group Best Practices

- Follow least-privilege access.
- Restrict SSH access to trusted IPs.
- Allow application traffic only through the ALB Security Group.
- Avoid unnecessary public access.

---

## 12. Scaling Policy Design

This module uses Amazon CloudWatch metrics to automatically adjust the number of EC2 instances in the Auto Scaling Group based on CPU utilization. It helps maintain application availability while optimizing resource usage and cost.

### 12.1 Scale Out Policy

| Parameter | Value |
|-----------|-------|
| **Condition** | CPU Utilization > 70% |
| **Action** | Add 1 EC2 instance |
| **Cooldown** | 300 seconds |

### 12.2 Scale In Policy

| Parameter | Value |
|-----------|-------|
| **Condition** | CPU Utilization < 30% |
| **Action** | Remove 1 EC2 instance |
| **Cooldown** | 300 seconds |

---


## 13. Best Practices Followed

| Best Practice | Description |
|--------------|-------------|
| **Reusable Module Design** | Designed as a reusable module that can be deployed across multiple applications and environments. |
| **Variable-Driven Configuration** | Uses input variables instead of hardcoded values for better flexibility and maintainability. |
| **Support for Existing AWS Resources** | Reuses existing infrastructure through Terraform data sources wherever possible. |
| **Least-Privilege Security** | Applies the principle of least privilege by allowing only the required network access. |
| **Standard Resource Tagging** | Uses consistent resource tags for better governance, cost tracking, and resource management. |
| **ALB Integration Support** | Supports integration with Application Load Balancer Target Groups for traffic distribution and health checks. |

---

## 14. References

| Reference | Description |
|-----------|-------------|
| **Terraform Documentation** | https://developer.hashicorp.com/terraform/docs |
| **Terraform AWS Provider** | https://registry.terraform.io/providers/hashicorp/aws/latest/docs |
| **Amazon EC2 Auto Scaling** | https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html |
| **Amazon EC2 Launch Templates** | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html |
| **Amazon CloudWatch Documentation** | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ |
| **AWS Security Groups** | https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html |
| **AWS Application Load Balancer** | https://docs.aws.amazon.com/elasticloadbalancing/latest/application/ |

---

## 15. Contact Information

| Name | Email |
|------|-------|
| Bhawna Dangarh | bhawna.dangarh.snaatak@mygurukulam.co |
