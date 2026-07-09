---
title: "Blog 1"
date: 2026-06-04
weight: 1
chapter: false
pre: " <b>3.1.</b> "
---

# Cloud Security with Amazon VPC Encryption Controls

In modern enterprise systems, encrypting data in transit is a critical requirement to meet security standards such as **HIPAA**, **PCI DSS**, **FedRAMP**, and **SOC 2**. However, as AWS infrastructure scales to hundreds or thousands of resources, identifying which traffic is encrypted and which resources are still transmitting data in plaintext becomes highly challenging.

To address this challenge, AWS has introduced **Amazon VPC Encryption Controls**—a new security feature that helps organizations monitor, control, and enforce encryption in transit between AWS resources within or across multiple VPCs in a Region.

## What is Amazon VPC Encryption Controls?

Amazon VPC Encryption Controls is a security feature that allows you to:
* Monitor the encryption status of network traffic within a VPC.
* Identify resources that still allow unencrypted data transmission.
* Enforce a policy that mandates data encryption across your entire AWS environment.
* Simplify the auditing process and meet compliance requirements.

This feature leverages two primary encryption mechanisms:

* **Application Layer Encryption** such as TLS.
* **Hardware-based encryption by the AWS Nitro System**, integrated into modern instance generations.

Beyond Amazon EC2, AWS has expanded Nitro's encryption capabilities to several other services, including:

* Application Load Balancer (ALB)
* Network Load Balancer (NLB)
* AWS Fargate
* Amazon EKS
* Amazon ECS
* Amazon RDS
* Amazon OpenSearch Service
* Amazon MSK
* AWS Transit Gateway

As a result, organizations can deploy encryption consistently at scale without the need to build a complex PKI system or rely on multiple third-party security solutions.

---

## Two Primary Operating Modes

### 1. Monitor Mode – Observe before Enforcing

AWS recommends that all existing VPCs start with **Monitor Mode**.

In this mode, VPC Encryption Controls **does not block traffic**, but merely records and evaluates the encryption status of connections.

A new data field named **encryption-status** is added to **VPC Flow Logs** with the following values:

| Value | Meaning |
| --- | --- |
| 0 | Unencrypted |
| 1 | Encrypted by AWS Nitro |
| 2 | Application layer encryption (TLS) |
| 3 | Nitro + TLS |
| No value | Unknown |

Through Flow Logs, administrators can quickly identify:

* Which data flows are protected.
* Which services still allow plaintext transmission.
* Which resources require an upgrade prior to enforcing encryption policies.

Additionally, AWS provides the following API:

```
GetVpcResourcesBlockingEncryptionEnforcement

```

This API helps list resources that do not meet encryption requirements before transitioning to Enforce Mode.

---

## Transitioning to a Fully Encrypted Environment
After evaluating the system using Monitor Mode, organizations can proceed to transition their infrastructure.

### Services Automatically Upgraded by AWS

AWS automatically transitions the underlying infrastructure to AWS Nitro for the following services:

* Application Load Balancer
* Network Load Balancer
* AWS Fargate
* Amazon EKS Control Plane

This process occurs with:

* Zero downtime.
* No configuration changes required.
* No manual intervention.

### Services Requiring Manual Upgrades

Certain resources still require customers to proactively transition them, including:

* Older generation EC2 instances
* Auto Scaling Groups
* Amazon RDS
* Amazon ElastiCache
* Amazon Redshift
* Amazon ECS using EC2
* Amazon OpenSearch Service
* Amazon EMR

If these resources are using instances that do not support Nitro, organizations must:

* Migrate to modern EC2 instance types that support the AWS Nitro System.
* Alternatively, deploy TLS at the application layer.

---

## Enforce Mode – Enforcing Encryption Policies

Once the evaluation and transition process is complete, organizations can activate **Enforce Mode**.

This is the phase where AWS actively enforces the security policy.

When Enforce Mode is enabled:

* You cannot launch older generation EC2 instances that do not support AWS Nitro.
* You cannot deploy resources that permit unencrypted traffic.
* Only encrypted connections are permitted to operate within the VPC.

As a result, all internal traffic is consistently protected without relying entirely on users correctly configuring TLS.

---

## Supported Exclusions

In practice, certain resources need to communicate outside the AWS infrastructure and cannot be fully managed by VPC Encryption Controls.

AWS allows you to configure **Exclusions** for resources such as:

* Internet Gateway
* NAT Gateway
* Egress-only Internet Gateway
* Virtual Private Gateway
* VPC Peering
* AWS Lambda in a VPC
* VPC Lattice
* Amazon EFS

These exclusions are fully documented during the auditing process, helping organizations demonstrate their compliance posture when undergoing audits.

---

## Benefits for Organizations

### Enhanced Security

* Ensures all internal traffic is encrypted.
* Mitigates the risk of data leakage during transmission.

### Simplified Operations

Organizations are relieved from:

* Building dedicated PKI systems.
* Managing large volumes of digital certificates.
* Deploying disparate security solutions.

### Support for Auditing and Compliance

VPC Encryption Controls provides:

* Encryption status reporting.
* VPC Flow Logs tailored for auditing.
* Evidence to satisfy standards such as HIPAA, PCI DSS, and FedRAMP.

### Leveraging AWS Nitro

Because the encryption is handled at the hardware level, there is virtually no performance impact on your applications.

---

## Conclusion
Amazon VPC Encryption Controls represents a significant step forward in simplifying network security on AWS. Instead of having to build complex monitoring and encryption control systems, organizations can utilize this built-in solution to:

* Monitor the encryption status of all network traffic.
* Identify security blind spots.
* Enforce encryption policies centrally.
* Meet strict compliance requirements.

If your organization is working towards certifications like **HIPAA**, **PCI DSS**, or **FedRAMP**, deploying **Monitor Mode** is the ideal first step to evaluate your infrastructure's security posture before transitioning to **Enforce Mode**.

---

## References

- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-encryp…mazon.com/vpc/latest/userguide/vpc-encryption-controls.html
- https://aws.amazon.com/blogs/aws/introducing-vpc-encryption…e-encryption-in-transit-within-and-across-vpcs-in-a-region/