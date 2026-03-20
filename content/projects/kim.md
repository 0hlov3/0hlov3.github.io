---
title: "KIM (Kommunikation im Medizinwesen)"
description: "Design and operation of a highly secure and scalable communication platform for the exchange of medical data within the German Telematikinfrastruktur (TI)."
tags: ["kubernetes", "iac", "healthcare", "distributed-systems", "high-availability", "identity", "secrets-management"]
external: "https://akquinet.com/gesundheitswesen/kim.html"
status: "active"
starts: "2020"
ends: "present"
---

> Secure, highly available email platform for healthcare communication within Germany’s Telematikinfrastruktur (gematik).

Design, implementation, and operation of a highly available and secure platform for **KIM (Kommunikation im Medizinwesen)**, enabling encrypted email communication between healthcare providers within the German Telematikinfrastruktur (TI) specified by gematik.

The platform was built to meet strict regulatory and security requirements while supporting growing scalability and performance requirements. A multi-cluster Kubernetes architecture was designed and provisioned from scratch to separate environments such as administration, development, reference, testing, and production in accordance with gematik specifications.

The platform integrates multiple distributed components, including a mail server based on Apache James, object storage, messaging systems, and distributed databases, forming a resilient and horizontally scalable architecture for handling sensitive medical data. Authentication and access to Kubernetes clusters are implemented using Pinniped in combination with Microsoft Entra ID to enable secure and centralized identity federation. Secrets management is handled via External Secrets integrated with Azure Key Vault, ensuring secure and dynamic provisioning of sensitive configuration data. Backup and disaster recovery strategies were implemented using Velero and Medusa, complemented by cross-site replication of object storage via MinIO to ensure data durability and resilience across environments.

## Key Responsibilities

- Design and implementation of a multi-cluster Kubernetes platform (kubeadm-based)
- Provisioning of infrastructure using Infrastructure as Code (Packer, Terraform, Ansible)
- Setup and operation of multiple isolated environments (admin, dev, reference, test, production)
- Deployment and integration of core platform components (mail server, storage, messaging, databases)
- Implementation of identity and access management using Pinniped and Microsoft Entra ID
- Implementation of secrets management using External Secrets and Azure Key Vault
- Implementation of backup and disaster recovery strategies (Velero, Medusa, MinIO replication)
- Ensuring high availability, scalability, and security of the overall system
- Monitoring, troubleshooting, and continuous optimization of platform operations

## Technologies & Methods

- Kubernetes (kubeadm-based cluster provisioning)
- Infrastructure as Code (Packer, Terraform, Ansible)
- Apache James (mail server)
- MinIO (object storage, cross-site replication)
- Cassandra (distributed database)
- RabbitMQ (messaging)
- Elasticsearch (logging & search)
- ArgoCD (GitOps)
- Observability (Prometheus, Grafana)
- Identity & Access Management (Pinniped, Microsoft Entra ID)
- Secrets Management (External Secrets, Azure Key Vault)
- Backup & Disaster Recovery (Velero, Medusa)
- High-availability and distributed system design

## Impact

- Enabled secure digital communication, replacing legacy workflows such as fax and postal delivery
- Delivered a scalable and resilient platform architecture aligned with strict healthcare regulations
- Established fully automated and reproducible infrastructure across all environments
- Implemented robust backup, disaster recovery, and cross-site replication strategies for critical healthcare data