---
title: "TIMRef (TI-Messenger Referenzimplementierung)"
description: "Design and operation of a highly available Kubernetes-based reference platform for secure real-time communication within the German Telematikinfrastruktur (TI)."
tags: ["kubernetes", "golang", "operator", "gitops", "observability", "security", "identity"]
github: "https://github.com/gematik/TI-M-Referenzimplementierung"
external: "https://ehealthblog.akquinet.de/ehealth-blog/blogbeitrag-details/ti-messenger-was-leistet-das-referenzsystem-von-akquinet"
status: "active"
starts: "2022"
ends: "present"
---

> Kubernetes-based reference platform for secure, interoperable real-time communication in Germany’s Telematikinfrastruktur (gematik).

Design, implementation, and operation of highly available Kubernetes clusters forming the core platform infrastructure for the **TI-Messenger reference implementation (TIMRef)** specified by gematik.

The platform was built to support secure, interoperable real-time communication based on modern messaging protocols, while complying with strict regulatory and security requirements of the healthcare sector. A fully automated Kubernetes environment was provisioned from scratch and operated across multiple environments.

A custom Kubernetes Operator was developed in Go using Kubebuilder to automate complex platform workflows. Platform logic was modeled declaratively using Custom Resource Definitions (CRDs), enabling standardized and reproducible operations directly via the Kubernetes API.

The platform integrates messaging services based on Matrix (Synapse), along with observability, security, and automation components, forming a scalable and resilient platform for real-time healthcare communication. Authentication and access to Kubernetes clusters were implemented using Pinniped in combination with Microsoft Entra ID to enable secure and centralized identity federation. Keycloak was integrated to model and validate multi-tenant authentication scenarios within the reference architecture.

## Key Responsibilities

- Design and operation of Kubernetes clusters (kubeadm-based) on vSphere
- Infrastructure provisioning using Packer, Terraform, and Ansible
- Integration of NSX (NCP) for Kubernetes networking
- Development of a custom Kubernetes Operator (Kubebuilder, Go)
- Modeling platform workflows using Custom Resource Definitions (CRDs)
- Implementation of GitOps-based deployments (ArgoCD)
- Development of CI/CD pipelines (GitLab CI) for build and release automation
- Implementation of end-to-end security concepts (ECC certificates, policy enforcement)
- Implementation of identity and access management (Pinniped, Microsoft Entra ID)
- Integration of Keycloak for multi-tenant authentication scenarios
- Setup of observability stack (OpenTelemetry, Jaeger, Prometheus, Grafana, logging)
- Incident, problem, and change management in a production environment
- Provisioning and operation of PostgreSQL clusters using CloudNativePG (CNPG)
- Implementation of backup and disaster recovery strategies (Velero, Barman Cloud)

## Technologies & Methods

- Kubernetes (kubeadm, multi-environment clusters)
- Go (operator development, custom services)
- Kubebuilder (operator framework)
- Matrix Synapse (real-time messaging)
- Infrastructure as Code (Packer, Terraform, Ansible)
- VMware vSphere & NSX (NCP networking)
- ArgoCD (GitOps)
- GitLab CI/CD
- Observability (OpenTelemetry, Jaeger, Prometheus, Grafana, Elasticsearch)
- Security (Kyverno, Falco, TLS/ECC certificates)
- Identity & Access Management (Pinniped, Microsoft Entra ID, Keycloak)
- Distributed systems and platform engineering
- PostgreSQL (CloudNativePG)
- Backup & Disaster Recovery (Velero, Barman Cloud)

## Impact

- Delivered a reference architecture for secure real-time communication within the healthcare ecosystem
- Enabled standardized and automated platform operations through Kubernetes-native abstractions (CRDs, Operator)
- Ensured compliance with strict regulatory and security requirements of the Telematikinfrastruktur
- Established a fully automated, reproducible, and scalable platform foundation for TI-Messenger deployments