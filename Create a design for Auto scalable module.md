# AWS Auto-Scalable Compute Infrastructure - Terraform Module

## Document Details

| Author         | Created    | Version | Last Updated By | Last Edited On | L0 Reviewer                       | L1 Reviewer | L2 Reviewer    |
| -------------- | ---------- | ------- | --------------- | -------------- | --------------------------------- | ----------- | -------------- |
| Bhawna Dangarh | 2026-05-26 | 1.0     | Bhawna Dangarh  | 2026-05-27     | Sharvari Khamkar / Tina Bhatnagar | Aman Raj    | Abhishek Dubey |

---

# 1. Objective

This Terraform module is designed to provision a **reusable, configurable, and production-grade auto-scalable compute infrastructure on AWS**.

The module helps in:

- Provisioning EC2 instances through **Launch Template**
- Managing instance scaling through **Auto Scaling Group (ASG)**
- Enabling **metric-based scaling** using CloudWatch alarms
- Integrating compute instances with **Security Groups** and **Load Balancer Target Groups**
- Supporting **existing infrastructure** through Terraform `data` sources
- Providing a reusable design that can be consumed by multiple applications like:
  - frontend
  - employee-api
  - salary-api
  - attendance-api
  - backend services

---

# 2. Scope

This module will:

- Provision **Launch Template**
- Provision **Auto Scaling Group**
- Configure **Scale Out** and **Scale In** policies
- Configure **CloudWatch alarms** for scaling triggers
- Support **existing VPC, subnets, IAM role, and security groups**
- Optionally create a **Security Group**
- Optionally attach the ASG to **ALB Target Groups**
- Expose outputs for reuse in higher-level infrastructure

---

# 3. Terraform Resources Used

| Resource | Purpose |
|----------|---------|
| `aws_launch_template` | Defines EC2 instance configuration |
| `aws_autoscaling_group` | Creates and manages scalable EC2 instances |
| `aws_autoscaling_policy` | Defines scale-out and scale-in actions |
| `aws_cloudwatch_metric_alarm` | Triggers scaling based on thresholds |
| `aws_security_group` | Optionally creates SG for application access |
| `aws_lb_target_group_attachment` | Optionally attaches instances to target group |
| `aws_iam_instance_profile` | Associates IAM role with EC2 instances |
| `aws_key_pair` *(optional external reference)* | Used for SSH access if required |

---

# 4. Terraform Data Sources Used

This module/design may use `data.tf` to fetch already existing AWS resources instead of creating everything from scratch.

| Data Source | Purpose |
|------------|---------|
| `data.aws_vpc` | Fetch existing VPC details |
| `data.aws_subnets` / `data.aws_subnet` | Fetch subnet details for ASG placement |
| `data.aws_security_group` | Fetch existing security group |
| `data.aws_lb_target_group` | Fetch existing ALB target group |
| `data.aws_iam_instance_profile` | Fetch existing IAM instance profile |
| `data.aws_ami` | Fetch latest or specific AMI |
| `data.aws_key_pair` | Fetch existing key pair if SSH access is needed |

---

# 5. Module Structure

<img width="331" height="314" alt="image" src="https://github.com/user-attachments/assets/7bc9fdd7-24e8-4512-8dba-3597e0044046" />

# 6. Resource Interdependency Flow

<img width="354" height="431" alt="image" src="https://github.com/user-attachments/assets/c8bb5ab1-b632-480d-a214-3700ab0101bf" />



## 7. File Description

| File               | Purpose                                            |
| ------------------ | -------------------------------------------------- |
| `provider.tf`      | Defines AWS provider configuration                 |
| `versions.tf`      | Defines Terraform and provider version constraints |
| `variables.tf`     | Contains all input variables                       |
| `data.tf`          | Fetches existing AWS resources                     |
| `main.tf`          | Contains main infrastructure resources             |
| `outputs.tf`       | Exposes important outputs                          |
| `terraform.tfvars` | Stores environment-specific variable values        |
| `README.md`        | Module documentation                               |
| `examples/basic/`  | Example implementation of the module               |

## 8. Input Variables

| Variable                     | Type           | Description                                         | Example                                |
| ---------------------------- | -------------- | --------------------------------------------------- | -------------------------------------- |
| `name`                       | `string`       | Prefix for all resources                            | `"employee-api"`                       |
| `environment`                | `string`       | Environment name                                    | `"dev"`                                |
| `application`                | `string`       | Application name                                    | `"employee-api"`                       |
| `owner`                      | `string`       | Resource owner                                      | `"Neha"`                               |
| `cost_center`                | `string`       | Cost allocation tag value                           | `"engineering"`                        |
| `vpc_id`                     | `string`       | Existing VPC ID                                     | `"vpc-123abc"`                         |
| `subnet_ids`                 | `list(string)` | List of subnets for ASG                             | `["subnet-1","subnet-2"]`              |
| `ami_id`                     | `string`       | EC2 AMI ID. If blank, can be fetched from `data.tf` | `"ami-123456"`                         |
| `ami_name_filter`            | `string`       | AMI name pattern if using data source               | `"my-app-ami-*"`                       |
| `instance_type`              | `string`       | EC2 instance type                                   | `"t3.medium"`                          |
| `key_name`                   | `string`       | Existing key pair name for SSH access               | `"dev-key"`                            |
| `iam_instance_profile_name`  | `string`       | IAM instance profile name                           | `"ec2-ssm-role"`                       |
| `min_size`                   | `number`       | Minimum number of instances in ASG                  | `2`                                    |
| `max_size`                   | `number`       | Maximum number of instances in ASG                  | `5`                                    |
| `desired_capacity`           | `number`       | Desired number of instances                         | `2`                                    |
| `health_check_type`          | `string`       | Health check type (`EC2` or `ELB`)                  | `"ELB"`                                |
| `health_check_grace_period`  | `number`       | Grace period for health check                       | `300`                                  |
| `create_security_group`      | `bool`         | Whether module should create SG                     | `true`                                 |
| `security_group_ids`         | `list(string)` | Existing SG IDs if not creating SG                  | `["sg-123"]`                           |
| `ingress_rules`              | `list(object)` | Ingress rules for SG                                | custom object                          |
| `egress_rules`               | `list(object)` | Egress rules for SG                                 | custom object                          |
| `associate_target_groups`    | `bool`         | Whether ASG should attach to ALB TG                 | `true`                                 |
| `target_group_arns`          | `list(string)` | List of ALB Target Group ARNs                       | `["arn:aws:elasticloadbalancing:..."]` |
| `user_data`                  | `string`       | EC2 bootstrap script                                | base64/user data                       |
| `enable_detailed_monitoring` | `bool`         | Enables detailed CloudWatch monitoring              | `true`                                 |
| `scale_up_cpu_threshold`     | `number`       | CPU threshold to scale out                          | `70`                                   |
| `scale_down_cpu_threshold`   | `number`       | CPU threshold to scale in                           | `30`                                   |
| `scale_up_adjustment`        | `number`       | Number of instances to add                          | `1`                                    |
| `scale_down_adjustment`      | `number`       | Number of instances to remove                       | `-1`                                   |
| `cooldown`                   | `number`       | Cooldown period after scaling                       | `300`                                  |
| `evaluation_periods`         | `number`       | Number of evaluation periods for CloudWatch alarm   | `2`                                    |
| `period`                     | `number`       | Alarm metric period in seconds                      | `120`                                  |
| `alarm_namespace`            | `string`       | CloudWatch namespace                                | `"AWS/EC2"`                            |
| `alarm_metric_name`          | `string`       | CloudWatch metric name                              | `"CPUUtilization"`                     |
| `tags`                       | `map(string)`  | Additional resource tags                            | `{}`                                   |

## 9. Instance Design

| Application Type     | Recommended Instance Type | Reason                                       |
| -------------------- | ------------------------- | -------------------------------------------- |
| Frontend / Nginx     | `t3.micro` / `t3.small`   | Low traffic and lightweight workload         |
| Small API            | `t3.small` / `t3.medium`  | Moderate CPU and memory requirement          |
| Java API             | `t3.medium` / `t3.large`  | JVM-based workload usually needs more memory |
| Python API           | `t3.small` / `t3.medium`  | Suitable for Flask/FastAPI type apps         |
| High Traffic Backend | `t3.large` / `m6i.large`  | Better performance for heavy traffic         |
| Batch Worker         | `t3.medium` / `c6i.large` | Better for compute-heavy tasks               |

## Recommended default for this module

For most internal APIs, a good starting point is:

instance_type = "t3.medium"
min_size = 2
desired_capacity = 2
max_size = 5

This gives:

high availability across subnets
minimum baseline capacity
room to scale during load increase

## 10. Security Group Design

Two supported approaches are available:

Approach 1: Use Existing Security Group

Pass existing security group IDs through:

```
security_group_ids = ["sg-1234567890"]
create_security_group = false
```
Approach 2: Create Security Group from Module

```
create_security_group = true
```
Example Ingress Rule Structure

```
ingress_rules = [
  {
    description = "Allow HTTP from ALB"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = []
    security_groups = ["sg-alb123"]
  },
  {
    description = "Allow SSH from bastion"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
    security_groups = []
  }
]
```
## 11. Security Group Best Practices

| Best Practice | Description |
|---------------|-------------|
| No hardcoded open public access | Avoid `0.0.0.0/0` unless absolutely required (e.g., public HTTP/HTTPS). |
| Use ALB SG → App SG access | Allow traffic from ALB Security Group to Application Security Group instead of public access. |
| Keep SSH restricted | Allow port 22 only from specific IPs (Bastion/Office IP), not public internet. |
| Least-privilege model | Only required ports and protocols should be allowed. |
| Separate SGs | Use different Security Groups for Load Balancer and Application for better control and security. |

---

## 12. Scaling Policy Design

### Approach: Metric-Based Auto Scaling using CloudWatch

| Parameter | Value |
|----------|-------|
| Metric Used | CPUUtilization |
| Namespace | AWS/EC2 |
| Scaling Type | Target Tracking / Step Scaling |
| Default Behavior | CPU-based automatic scaling |

---

## 13.1 Scale Out Policy

### Condition

| Condition Parameter | Value |
|--------------------|-------|
| Metric | CPUUtilization |
| Threshold | > 70% |
| Evaluation Periods | 2 |
| Period Duration | 120 seconds |
| Total Observation Time | 240 seconds |

### Action

| Action | Value |
|--------|-------|
| Scaling Adjustment | +1 instance |
| Cooldown | 300 seconds (default) |

### Example

| Scenario | Result |
|----------|--------|
| Current Desired Capacity | 2 |
| CPU > 70% for 240 sec | Scale Out Triggered |
| New Desired Capacity | 3 |
| Continued High Load | Scales to 4, 5... until `max_size` |

---

## 13.2 Scale In Policy

### Condition

| Condition Parameter | Value |
|--------------------|-------|
| Metric | CPUUtilization |
| Threshold | < 30% |
| Evaluation Periods | 2 |
| Period Duration | 120 seconds |
| Total Observation Time | 240 seconds |

### Action

| Action | Value |
|--------|-------|
| Scaling Adjustment | -1 instance |
| Cooldown | 300 seconds (default) |

### Example

| Scenario | Result |
|----------|--------|
| Current Desired Capacity | 4 |
| CPU < 30% for 240 sec | Scale In Triggered |
| New Desired Capacity | 3 |
| Continued Low Load | Scales down to 2 → 1 → until `min_size` |

## 13.3 Scaling Policy Conditions Summary

| Policy    | Metric         | Condition         | Evaluation          | Action                |
| --------- | -------------- | ----------------- | ------------------- | --------------------- |
| Scale Out | CPUUtilization | `> 70%`           | 2 periods × 120 sec | `+1 instance`         |
| Scale In  | CPUUtilization | `< 30%`           | 2 periods × 120 sec | `-1 instance`         |
| Cooldown  | N/A            | After scale event | 300 sec             | Prevent rapid scaling |

## 13.4 Extensible Scaling Conditions

In future, this module can be extended for:

Memory utilization
Request count per target
Network in/out
Queue depth
Custom application metrics

## 14. Assumptions

| Assumption             | Description                                           |
| ---------------------- | ----------------------------------------------------- |
| VPC                    | Already created outside this module                   |
| Subnets                | Already available and passed as input                 |
| AMI                    | Either passed directly or fetched through data source |
| IAM Role               | IAM instance profile exists already                   |
| Target Group           | Optional and may already exist                        |
| DNS                    | Managed separately                                    |
| Listener Rules         | Managed separately                                    |
| Application Deployment | Managed separately through CI/CD                      |

## 15. Limitations

| Limitation            | Description                        |
| --------------------- | ---------------------------------- |
| Lifecycle Hooks       | Not implemented currently          |
| Spot Instances        | Not implemented currently          |
| Warm Pools            | Not implemented currently          |
| Instance Refresh      | Not implemented currently          |
| Mixed Instance Policy | Not implemented currently          |
| Memory-based scaling  | Requires custom CloudWatch metrics |

## 16. Future Enhancements

| Feature                | Description                      |
| ---------------------- | -------------------------------- |
| Mixed Instances Policy | Support On-Demand + Spot         |
| Scheduled Scaling      | Scale based on time              |
| Warm Pools             | Faster instance readiness        |
| Instance Refresh       | Rolling updates without downtime |
| Multi-Metric Scaling   | CPU + memory + ALB request count |
| Lifecycle Hooks        | Graceful startup/shutdown        |
| EBS customization      | Volume type and size support     |
| HTTPS-ready bootstrap  | Support secure app deployment    |

## 17. Best Practices Followed

| Best Practice | Description |
|---------------|-------------|
| Reusable Module Design | The module is designed in a reusable way so it can be used for multiple applications and environments. |
| No Hardcoded Environment Values | All important values are passed through variables instead of hardcoding them directly in Terraform files. |
| Support for Existing Infrastructure using `data.tf` | Existing AWS resources like VPC, subnets, IAM profile, and security groups can be fetched using data sources. |
| Separate Input, Output, and Logic Files | Different Terraform files are used for variables, outputs, data sources, and main resource logic to keep the code clean. |
| Least-Privilege Security Approach | Security group rules are designed to allow only the required access. |
| Tagging Standard for Billing and Governance | Standard tags like `Application`, `Owner`, `Environment`, and `CostCenter` are used for better tracking and billing. |
| Scalable and Production-Oriented ASG Design | The Auto Scaling Group is designed to support minimum, desired, and maximum capacity for production-style workloads. |
| Load Balancer Integration Support | The module supports integration with ALB target groups for traffic distribution and health checks. |
| Easy Extension for Future Use Cases | The design can be extended later for features like warm pools, mixed instances, scheduled scaling, and instance refresh. |


## 10. Contact Information

| Name           | Email                                                                                 |
| -------------- | ------------------------------------------------------------------------------------- |
| Bhawna Dangarh | [bhawna.dangarh.snaatak@mygurukulam.co](mailto:bhawna.dangarh.snaatak@mygurukulam.co) |
