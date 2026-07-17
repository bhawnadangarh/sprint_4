
# AWS Auto-Scalable Compute Infrastructure - Terraform Module

<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/876d2897-8dcd-4265-8134-df2826d364b6" />


## Document Details

| Author         | Created    | Version | Last Updated By | Last Edited On | L0 Reviewer                       | L1 Reviewer | L2 Reviewer    |
| -------------- | ---------- | ------- | --------------- | -------------- | --------------------------------- | ----------- | -------------- |
| Bhawna Dangarh | 2026-07-17 | 1.0     | Bhawna Dangarh  | 2026-07-18    | Sharvari Khamkar / Tina Bhatnagar | Aman Raj    | Abhishek Dubey |

# 1. Objective

This Terraform module provisions a reusable AWS Auto Scaling infrastructure. It creates a Launch Template, Auto Scaling Group (ASG), CloudWatch-based scaling policies, and optionally integrates with Security Groups and ALB Target Groups. The module is reusable across multiple applications and environments.

# 2. Scope

- Create Launch Template
- Create Auto Scaling Group
- Configure CPU-based auto scaling
- Support existing AWS resources using data sources
- Optionally attach ASG to ALB Target Groups
- Export outputs for reuse

# 3. Terraform Resources Used

| Resource | Purpose |
|----------|---------|
| aws_launch_template | Defines EC2 configuration |
| aws_autoscaling_group | Creates and manages scalable EC2 instances |
| aws_autoscaling_policy | Configures scaling actions |
| aws_cloudwatch_metric_alarm | Triggers scaling based on CPU |
| aws_security_group | Creates Security Group (optional) |
| aws_lb_target_group_attachment | Attaches instances to Target Group |
| aws_iam_instance_profile | Associates IAM role with EC2 |

# 4. Terraform Data Sources Used

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

# 5. Module Structure

> Add module structure diagram here.

# 6. Resource Interdependency Flow

> Add resource flow diagram here.

# 7. File Description

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

# 8. Input Variables

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

# 9. Instance Design

Choose the instance type based on your application's CPU and memory requirements. Lightweight services can use smaller instances, while Java or high-traffic applications may require larger instance types.

# 10. Security Group Design

The module supports two approaches:

- Use an existing Security Group.
- Create a new Security Group through the module.

Ingress and egress rules are configurable using input variables.

# 11. Security Group Best Practices

- Follow least-privilege access.
- Restrict SSH access to trusted IPs.
- Allow application traffic only through the ALB Security Group.
- Avoid unnecessary public access.

# 12. Scaling Policy Design

Auto Scaling is based on CloudWatch CPU utilization metrics. The module automatically scales instances up or down based on CPU thresholds to maintain availability and optimize resource usage.

# 13.1 Scale Out Policy

- Condition: CPU Utilization > 70%
- Action: Add 1 EC2 instance
- Cooldown: 300 seconds

# 13.2 Scale In Policy

- Condition: CPU Utilization < 30%
- Action: Remove 1 EC2 instance
- Cooldown: 300 seconds

# 13.3 Scaling Policy Summary

| Policy | Condition | Action |
|--------|-----------|--------|
| Scale Out | CPU > 70% | Add 1 instance |
| Scale In | CPU < 30% | Remove 1 instance |
| Cooldown | 300 seconds | Prevent rapid scaling |

# 13.4 Extensible Scaling Conditions

The module can be extended to support memory utilization, request count, network traffic, and custom application metrics.

# 14. Assumptions

- Existing VPC and subnets are available.
- IAM Instance Profile already exists.
- AMI is provided or fetched using data sources.
- Application deployment is managed separately.

# 15. Limitations

- Lifecycle Hooks are not supported.
- Spot Instances are not included.
- Memory-based scaling requires custom CloudWatch metrics.

# 16. Future Enhancements

Future improvements may include:

- Scheduled Scaling
- Spot Instances
- Warm Pools
- Instance Refresh
- Multi-Metric Scaling

# 17. Best Practices Followed

- Reusable module design
- Variable-driven configuration
- Support for existing AWS resources
- Least-privilege security
- Standard resource tagging
- ALB integration support

# 18. Contact Information

| Name | Email |
|------|-------|
| Bhawna Dangarh | bhawna.dangarh.snaatak@mygurukulam.co |
