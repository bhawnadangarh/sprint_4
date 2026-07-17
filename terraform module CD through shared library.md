# Terraform Module CD — Shared Library (Documentation)

## Document Details

| Author | Created | Version | Last Updated By | Last Edited On | L0 Reviewer | L1 Reviewer | L2 Reviewer |
|--------|---------|---------|-----------------|----------------|-------------|-------------|-------------|
| Bhawna Dangarh | 2026-07-17 | 1.0 | Bhawna Dangarh | 2026-07-18 | Sharvari Khamkar / Tina Bhatnagar | Aman Raj | Abhishek Dubey |

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [What is CD Shared Library?](#2-what-is-cd-shared-library)
3. [Why We Use CD Shared Library?](#3-why-we-use-cd-shared-library)
4. [Prerequisites](#4-prerequisites)
5. [Supported Functions](#5-supported-functions)
   - [terraformApply()](#1-terraformapply)
   - [terraformDestroy()](#2-terraformdestroy)
6. [Jenkins Usage Example](#6-jenkins-usage-example)
7. [Inputs and Outputs](#7-inputs-and-outputs)
8. [How the Library Is Structured](#8-how-the-library-is-structured)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

---

# 1. Introduction

Terraform is used to create and manage cloud infrastructure. Instead of writing the same deployment code in every Jenkins pipeline, we use a **CD Shared Library**. It stores common deployment functions in one place, making pipelines easier to reuse and maintain.

---

# 2. What is CD Shared Library?

A CD Shared Library is a common repository that stores reusable Jenkins pipeline code for Terraform deployment. Instead of writing the same deployment steps in every Jenkinsfile, the code is written once and reused across multiple projects.

This library provides reusable functions like **terraformApply()** and **terraformDestroy()** to deploy and remove infrastructure.

---

# 3. Why We Use CD Shared Library?

- Avoids writing the same deployment code in every project.
- Keeps all deployment pipelines consistent.
- Makes pipelines easier to maintain.
- Reduces duplicate code.
- Allows reusable deployment functions.

---

# 4. Prerequisites

Before using this library, make sure:

- Terraform is installed.
- Jenkins Shared Library is configured.
- Cloud credentials are configured.
- Terraform backend is configured.
- Terraform code is available in the Jenkins workspace.

---

# 5. Supported Functions

The Terraform CD Shared Library provides the following functions.

---

## 1. terraformApply()

Deploys or updates the Terraform infrastructure.

**Command**

```bash
terraform apply -auto-approve
```

---

## 2. terraformDestroy()

Deletes the Terraform infrastructure.

**Command**

```bash
terraform destroy -auto-approve
```

---

# 6. Jenkins Usage Example

```groovy
@Library('terraform-shared-lib') _

pipeline {
    agent any

    stages {

        stage('Terraform Apply') {
            steps {
                script {
                    terraformApply(
                        workingDir: 'terraform',
                        varFile: 'dev.tfvars'
                    )
                }
            }
        }

    }
}
```

---

# 7. Inputs and Outputs

## Inputs

| Parameter | Description |
|-----------|-------------|
| `workingDir` | Path to the Terraform module |
| `varFile` | Terraform variable file |
| `extraArgs` | Additional Terraform arguments |

## Outputs

- Shows the Terraform output in the Jenkins console.
- Deploys or destroys the infrastructure.
- Stops the pipeline if any command fails.

---

# 8. How the Library Is Structured

The Terraform CD Shared Library is divided into different folders to keep the code organized.

- **src/** contains reusable Groovy classes.
- **vars/** contains global pipeline functions.
- **resources/** stores supporting files used by the library.

```text
terraform-shared-lib/
│
├── src/
│   └── org/
│       └── terraform/
│           ├── TerraformApply.groovy
│           └── TerraformDestroy.groovy
│
├── vars/
│   └── terraform.groovy
│
├── resources/
│
└── README.md
```

---

# 9. Conclusion

The Terraform CD Shared Library makes Terraform deployments simple and reusable. It provides common deployment functions that can be used across multiple Jenkins pipelines, helping teams maintain consistent and reliable deployment workflows.

---

# 10. Contact Information

| Name | Email |
|------|-------|
| Bhawna Dangarh | bhawna.dangarh.snaatak@mygurukulam.co |

---

# 11. References

| Description | Link |
|------------|------|
| Terraform Documentation | https://developer.hashicorp.com/terraform/docs |
| Terraform CLI | https://developer.hashicorp.com/terraform/cli |
| Jenkins Shared Library | https://www.jenkins.io/doc/book/pipeline/shared-libraries/ |

---
