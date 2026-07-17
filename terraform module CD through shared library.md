# Terraform Module CI/CD — Shared Library (DOC)


## Document Details

| Author         | Created    | Version | Last Updated By | Last Edited On | L0 Reviewer                       | L1 Reviewer | L2 Reviewer    |
| -------------- | ---------- | ------- | --------------- | -------------- | --------------------------------- | ----------- | -------------- |
| Bhawna Dangarh | 2026-07-17 | 1.0     | Bhawna Dangarh  | 2026-07-18    | Sharvari Khamkar / Tina Bhatnagar | Aman Raj    | Abhishek Dubey |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is CI Shared Library?](#2-what-is-ci-shared-library)
3. [Why We Use CI Shared Library?](#3-why-we-use-ci-shared-library)
4. [Supported Functions](#4-supported-functions)
   - [terraformInit()](#1-terraforminit)
   - [terraformValidate()](#2-terraformvalidate)
   - [terraformPlan()](#3-terraformplan)
   - [terraformCheckov()](#4-terraformcheckov)
   - [terraformInfraCost()](#5-terraforminfracost)
5. [Inputs and Outputs](#5-inputs-and-outputs)
6. [How the Library Is Structured](#6-how-the-library-is-structured)
7. [Conclusion](#7-conclusion)
8. [Contact Information](#8-contact-information)
9. [References](#9-references)

---

## 1. Introduction

Modern DevOps environments require a consistent and standardized approach to managing Terraform across multiple projects. A shared CI library centralizes reusable operations like `init`, `validate`, `plan`, security scanning, and cost estimation — reducing duplication, improving efficiency, and ensuring uniform, maintainable pipelines.

This library follows the **Jenkins `src/` format**, which uses Groovy classes instead of simple scripts, giving better code organization, reusability, and testability.

---

## 2. What is CI Shared Library?

A CI shared library centralizes reusable pipeline logic, enabling multiple projects to reuse standardized build, validation, testing, and deployment workflows without duplicating code.

- Tools like **Jenkins** allow pipelines to call shared functions from a common repository
- The **`src/` format** organizes these functions as proper Groovy classes under a package structure
- This improves consistency, maintainability, and scalability across teams

---

## 3. Why We Use CI Shared Library?

- Eliminates duplicate pipeline logic across multiple projects
- Ensures consistent Terraform standards and best practices across all projects
- A single update to the library benefits all projects immediately
- Reduces human error from copy-pasting pipeline code
- The `src/` format adds proper class-based structure, making the library easier to unit test

---

## 4. Supported Functions

> All functions live under the package `org.terraform` inside `src/`.  
> Each function is a Groovy class with a `call(Map config)` method.

---

### 1. `terraformInit()`

**Class:** `org.terraform.TerraformInit`

**Purpose:** Initializes the Terraform working directory by downloading required providers, modules, and backend configurations.

**Why it matters:** Terraform needs provider plugins (like AWS, Azure, GCP) and modules before it can understand or process your code. Without initialization, Terraform cannot validate, plan, or apply infrastructure.

**Behind the scenes:**

```bash
terraform init
```

**Usage in Jenkinsfile:**

```groovy
@Library('terraform-shared-lib') _
import org.terraform.TerraformInit

def tf = new TerraformInit()
tf.call(workingDir: 'modules/vpc')
```

---

### 2. `terraformValidate()`

**Class:** `org.terraform.TerraformValidate`

**Purpose:** Checks whether the Terraform code is syntactically correct and properly configured.

**Why `init` is needed first:** Validation depends on provider plugins and modules downloaded during `terraform init`. Without them, Terraform cannot verify resource types, arguments, or references properly.

**Example:** If your code uses `aws_instance`, Terraform must first download the AWS provider to understand whether that resource is valid.

**Behind the scenes:**

```bash
terraform validate
```

**Usage in Jenkinsfile:**

```groovy
@Library('terraform-shared-lib') _
import org.terraform.TerraformValidate

def tf = new TerraformValidate()
tf.call(workingDir: 'modules/vpc')
```

---

### 3. `terraformPlan()`

**Class:** `org.terraform.TerraformPlan`

**Purpose:** Shows what changes Terraform will make — create, update, or destroy resources.

**Why `init` is needed first:** Planning requires provider details and backend configuration to compare your code with actual infrastructure state. Without `terraform init`, Terraform cannot generate an accurate execution plan.

**Example:** Before planning an EC2 instance creation, Terraform must first know how AWS resources work through the AWS provider.

**Behind the scenes:**

```bash
terraform plan -var-file=<varFile>
```

**Usage in Jenkinsfile:**

```groovy
@Library('terraform-shared-lib') _
import org.terraform.TerraformPlan

def tf = new TerraformPlan()
tf.call(workingDir: 'modules/vpc', varFile: 'envs/dev.tfvars')
```

---

### 4. `terraformCheckov()`

**Class:** `org.terraform.TerraformCheckov`

**Purpose:** Scans Terraform code for security misconfigurations and compliance violations **before** any infrastructure is created.

**Why it matters:** Checkov is a static analysis tool. It reads your `.tf` files and checks them against hundreds of security rules — catching issues before they reach production.

**What it checks (examples):**

- S3 buckets without encryption or public access blocks
- Security groups with unrestricted `0.0.0.0/0` access
- IAM policies with wildcard (`*`) permissions
- Missing logging or monitoring configurations

**Behind the scenes:**

```bash
checkov -d .
```

**Output:** A pass/fail report per resource. The build **fails** if any HIGH or CRITICAL severity check fails (unless `softFail: true`).

**Usage in Jenkinsfile:**

```groovy
@Library('terraform-shared-lib') _
import org.terraform.TerraformCheckov

def ck = new TerraformCheckov()
ck.call(workingDir: 'modules/vpc', softFail: false)
```

---

### 5. `terraformInfraCost()`

**Class:** `org.terraform.TerraformInfraCost`

**Purpose:** Estimates the **monthly cloud cost** of your Terraform changes before they are applied.

**Why it matters:** Without cost visibility in CI, engineers can accidentally provision expensive resources (large EC2 instances, NAT Gateways, data transfer) and only find out on the monthly bill.

**What it shows (examples):**

- Cost of new resources being created
- Cost difference compared to existing infrastructure
- Per-resource monthly estimate breakdown

**Behind the scenes:**

```bash
infracost breakdown --path .
```

**Output:** A cost breakdown table printed to the Jenkins console. Optionally posts a cost diff comment on the pull request when an API key is provided.

**Usage in Jenkinsfile:**

```groovy
@Library('terraform-shared-lib') _
import org.terraform.TerraformInfraCost

def ic = new TerraformInfraCost()
ic.call(workingDir: 'modules/vpc', infracostApiKey: credentials('infracost-api-key'))
```

---

## 5. Inputs and Outputs

### Inputs (passed as a `Map` to each function's `call()` method)

| Parameter          | Required | Used By                              | Default | What it does                                              |
|--------------------|----------|--------------------------------------|---------|-----------------------------------------------------------|
| `workingDir`       | Yes      | All functions                        | —       | Path to your Terraform module folder                      |
| `varFile`          | No       | `plan`, `checkov`, `infracost`       | `""`    | Path to a `.tfvars` file                                  |
| `backendConfig`    | No       | `init`                               | `""`    | Path to a backend config file for init                    |
| `extraArgs`        | No       | All functions                        | `""`    | Any extra flags to pass to the tool                       |
| `softFail`         | No       | `checkov`                            | `false` | If `true`, Checkov failures warn but don't break the build|
| `infracostApiKey`  | No       | `infracost`                          | `""`    | API key for Infracost Cloud (for PR cost comments)        |

### Outputs

Each function:

- **Prints** tool output to the Jenkins console log
- **Returns exit code** — `0` means success, anything else means failure
- **Fails the build** automatically if the tool exits with an error (except when `softFail: true`)

---

## 6. How the Library Is Structured

The library uses the **Jenkins `src/` format**. This means all logic lives as Groovy classes under a package, not as loose scripts.

```
terraform-shared-lib/
│
├── src/
│   └── org/
│       └── terraform/
│           ├── TerraformInit.groovy         # terraform init
│           ├── TerraformValidate.groovy     # terraform validate
│           ├── TerraformPlan.groovy         # terraform plan
│           ├── TerraformCheckov.groovy      # checkov security scan
│           └── TerraformInfraCost.groovy    # infracost estimation
│
├── vars/
│   └── terraform.groovy                    # optional: single entry point wrapper
│
├── test/
│   └── org/
│       └── terraform/
│           ├── TerraformInitTest.groovy
│           ├── TerraformValidateTest.groovy
│           ├── TerraformPlanTest.groovy
│           ├── TerraformCheckovTest.groovy
│           └── TerraformInfraCostTest.groovy
│
└── README.md
```

### Why `src/` format over `vars/`?

| Point          | `vars/` format              | `src/` format                          |
|----------------|-----------------------------|----------------------------------------|
| Structure      | Flat scripts                | Groovy classes with packages           |
| Reusability    | Limited                     | Classes can extend and import each other |
| Unit Testing   | Hard to test                | Fully testable with JUnit/Spock        |
| Code Quality   | No OOP                      | Supports OOP principles                |
| Best for       | Simple, quick scripts       | Production-grade shared libraries      |

---

## 7. Conclusion

The Terraform Module CI/CD Shared Library provides a standardized, reusable, and maintainable framework for executing core Terraform operations within Jenkins pipelines. Migrated to the **`src/` format**, it now supports proper class-based structure, unit testing, and clean package organization. With Checkov for security scanning and Infracost for cost estimation added, the library covers the full CI quality gate — syntax, validation, security, and cost — before any infrastructure change reaches production.

---

## 8. Contact Information

| Name | Email |
|------|-------|
| Bhawna Dangarh | bhawna.dangarh.snaatak@mygurukulam.co |

---

## 9. References

| Description                           | Link                                                                                          |
|---------------------------------------|-----------------------------------------------------------------------------------------------|
| Terraform Module CI/CD Shared Library | [GitHub Repo](https://github.com/your-repo-path/terraform-shared-lib/README.md)             |
| Terraform Official Documentation      | [Terraform Docs](https://developer.hashicorp.com/terraform/docs)                             |
| Jenkins Shared Library (`src/` format)| [Jenkins Shared Library](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)         |
| Terraform CLI Commands                | [Terraform CLI](https://developer.hashicorp.com/terraform/cli)                               |
| Checkov Documentation                 | [Checkov Docs](https://www.checkov.io/1.Welcome/What-is-Checkov.html)                       |
| Infracost Documentation               | [Infracost Docs](https://www.infracost.io/docs/)                                             |
| CI/CD Best Practices                  | [CI/CD Guide](https://www.redhat.com/en/topics/devops/what-is-ci-cd)                        |

---
