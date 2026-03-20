---
title: "TI-Messenger (Product Platform)"
description: "Operation and continuous development of a production-grade, Kubernetes-based messaging platform for secure real-time communication in the German healthcare ecosystem."
tags: ["kubernetes", "platform-engineering", "golang", "gitops", "security", "observability", "identity"]
external: "https://akquinet.com/gesundheitswesen/tim.html"
status: "active"
starts: "2022"
ends: "present"
---

> Production-grade platform for secure, scalable, and interoperable real-time communication within Germany’s Telematikinfrastruktur.

Operation and continuous development of a **TI-Messenger (TIM) product platform** built on top of the gematik reference architecture, providing secure, interoperable real-time communication services for healthcare providers within the Telematikinfrastruktur (TI).

The platform was designed for production workloads with a strong focus on reliability, scalability, and operational excellence. Multiple Kubernetes clusters were provisioned and operated across environments, forming the backbone of a highly available messaging system based on Matrix (Synapse).

A custom Kubernetes Operator written in Go (Kubebuilder) was developed and extended to manage complex platform workflows and lifecycle operations. Platform behavior was modeled declaratively using Custom Resource Definitions (CRDs), enabling standardized and automated operations at scale.

Stateful services such as PostgreSQL clusters were operated using CloudNativePG (CNPG), including automated backup and disaster recovery strategies using Velero and Barman Cloud. The platform also incorporates a service mesh (Linkerd) with mutual TLS (mTLS) to secure inter-service communication. Authentication and access to Kubernetes clusters are managed using Pinniped in combination with Microsoft Entra ID, enabling secure and centralized identity federation. For customer-facing multi-tenancy, Keycloak is used to manage tenant isolation and authentication flows.

A comprehensive observability stack (OpenTelemetry, Jaeger, Prometheus, Grafana, OpenSearch) was implemented to ensure full system visibility, traceability, and efficient incident response.

The platform integrates messaging services based on Matrix (Synapse), along with stateful components such as PostgreSQL clusters managed via CloudNativePG (CNPG), as well as observability, security, and automation components, forming a scalable and resilient platform for real-time healthcare communication, supporting multi-tenant deployments and horizontally scalable messaging workloads.

## Key Responsibilities

- Operation of a production-grade Kubernetes platform (kubeadm-based) on vSphere
- Infrastructure provisioning using Packer, Terraform, and Ansible
- Integration of NSX (NCP) for Kubernetes networking
- Development and extension of a custom Kubernetes Operator (Kubebuilder, Go)
- Modeling platform workflows using Custom Resource Definitions (CRDs)
- Implementation of GitOps-based deployments (ArgoCD)
- Development of CI/CD pipelines (GitLab CI) for build and release automation
- Implementation of end-to-end security concepts (ECC certificates, mTLS via Linkerd, policy enforcement)
- Implementation of identity and access management using Pinniped and Microsoft Entra ID
- Design and operation of multi-tenant authentication architecture using Keycloak
- Setup and operation of observability stack (OpenTelemetry, Jaeger, Prometheus, Grafana, OpenSearch)
- Provisioning and operation of PostgreSQL clusters using CloudNativePG (CNPG)
- Implementation of backup and disaster recovery strategies (Velero, Barman Cloud)
- Incident, problem, and change management in a production environment
- Performance optimization and continuous platform improvement in collaboration with development teams

## Technologies & Methods

- Kubernetes (kubeadm, multi-environment clusters)
- Go (operator development, custom services)
- Kubebuilder (operator framework)
- Matrix Synapse (real-time messaging)
- Infrastructure as Code (Packer, Terraform, Ansible)
- VMware vSphere & NSX (NCP networking)
- ArgoCD (GitOps)
- GitLab CI/CD
- Service Mesh (Linkerd, mTLS)
- Observability (OpenTelemetry, Jaeger, Prometheus, Grafana, OpenSearch)
- Security (Kyverno, Falco, TLS/ECC certificates)
- Identity & Access Management (Pinniped, Microsoft Entra ID, Keycloak)
- PostgreSQL (CloudNativePG)
- Backup & Disaster Recovery (Velero, Barman Cloud)
- Distributed systems and platform engineering

## Impact

- Delivered a production-grade messaging platform for secure healthcare communication
- Enabled scalable and reliable real-time communication based on Matrix within a regulated environment
- Established Kubernetes-native automation through Operators and CRDs, reducing operational complexity
- Implemented end-to-end security including mTLS and policy enforcement aligned with regulatory requirements
- Ensured high platform reliability and observability for production workloads