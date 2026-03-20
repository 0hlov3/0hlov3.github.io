---
title: "KIM (Kommunikation im Medizinwesen)"
description: "Design and operation of a highly secure and scalable communication platform for the exchange of medical data within the German Telematikinfrastruktur (TI)."
tags: ["kubernetes", "iac", "healthcare", "distributed-systems", "high-availability"]
external: "https://akquinet.com/gesundheitswesen/kim.html"
status: "active"
starts: "2020"
ends: "present"
---

> Secure, highly available email platform for healthcare communication within Germany’s Telematikinfrastruktur (gematik).

Design, implementation, and operation of a highly available and secure platform for **KIM (Kommunikation im Medizinwesen)**, enabling encrypted email communication between healthcare providers within the German Telematikinfrastruktur (TI) specified by gematik.

The platform was built to meet strict regulatory and security requirements while supporting growing scalability and performance requirements. A multi-cluster Kubernetes architecture was designed and provisioned from scratch to separate environments such as administration, development, reference, testing, and production in accordance with gematik specifications.

The platform integrates multiple distributed components, including a mail server based on Apache James, object storage, messaging systems, and distributed databases, forming a resilient and horizontally scalable architecture for handling sensitive medical data.
## Key Responsibilities

- Design and implementation of a multi-cluster Kubernetes platform (kubeadm-based)
- Provisioning of infrastructure using Infrastructure as Code (Packer, Terraform, Ansible)
- Setup and operation of multiple isolated environments (admin, dev, reference, test, production)
- Deployment and integration of core platform components (mail server, storage, messaging, databases)
- Ensuring high availability, scalability, and security of the overall system
- Monitoring, troubleshooting, and continuous optimization of platform operations

## Technologies & Methods

- Kubernetes (kubeadm-based cluster provisioning)
- Infrastructure as Code (Packer, Terraform, Ansible)
- Apache James (mail server)
- MinIO (object storage)
- Cassandra (distributed database)
- RabbitMQ (messaging)
- Elasticsearch (logging & search)
- ArgoCD (GitOps)
- Observability (Prometheus, Grafana)
- High-availability and distributed system design

## Impact

- Enabled secure digital communication, replacing legacy workflows such as fax and postal delivery
- Delivered a scalable and resilient platform architecture aligned with strict healthcare regulations
- Established fully automated and reproducible infrastructure across all environments