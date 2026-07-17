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

A CI Shared Library is a common repository that stores reusable pipeline code for Jenkins. Instead of writing the same pipeline steps in every Jenkinsfile, you can write them once in the shared library and reuse them across multiple projects. In this Terraform Shared Library, common tasks such as `terraform init`, `terraform validate`, `terraform plan`, Checkov security scanning, and Infracost estimation are available as reusable functions. This helps keep pipelines consistent, reduces duplicate code, and makes them easier to maintain.

---

## 3. Why We Use CI Shared Library?

- Avoids writing the same pipeline code in every project.
- Ensures all Terraform pipelines follow the same process.
- A change made in the shared library is automatically available to all projects using it.
- Reduces copy-paste errors and makes pipelines easier to maintain.
- Provides a well-structured and reusable codebase using the `src/` format.
  
---

## 4. Supported Functions

The Terraform CI Shared Library provides the following reusable functions:

---

### 1. `terraformInit()`

Initializes the Terraform working directory by downloading the required providers, modules, and backend configuration. It prepares Terraform so other commands can run successfully.

**Command:**
```bash
terraform init
```

---

### 2. `terraformValidate()`

Checks whether the Terraform configuration is valid and free from syntax errors. It helps identify issues before planning or deploying the infrastructure.

**Command:**
```bash
terraform validate
```

---

### 3. `terraformPlan()`

Shows the resources that Terraform will create, update, or delete before deployment. This helps you review the changes before they are applied.

**Command:**
```bash
terraform plan -var-file=<varFile>
```

---

### 4. `terraformCheckov()`

Scans Terraform code for security issues and configuration mistakes. It helps identify security risks before deploying the infrastructure.

**Command:**
```bash
checkov -d .
```

---

### 5. `terraformInfraCost()`

Estimates the monthly cost of the Terraform infrastructure before deployment. It helps you understand the expected cloud cost before applying any changes.

**Command:**
```bash
infracost breakdown --path .
```


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

The Terraform CI Shared Library is organized into different directories to keep the code clean and easy to maintain. Each directory has a specific purpose, such as storing reusable Groovy classes, global pipeline functions, and supporting resource files.

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/0ff29e8e-349d-4f8d-b0bc-f4a14d06554a" />

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
