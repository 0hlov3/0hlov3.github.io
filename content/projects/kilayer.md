---
title: "KI Layer"
description: "Design and operation of a secure, bare-metal Kubernetes platform for GPU-accelerated and confidential artificial intelligence workloads in an enterprise environment."
tags: [ "Kubernetes", "Talos Linux", "Platform Engineering", "GPU", "GitOps", "Security", "Observability"]
status: active
starts: 2026
ends: present
---

> Secure, GPU-enabled Kubernetes platform for confidential artificial intelligence workloads on bare-metal infrastructure.

The KI Layer is a Kubernetes-based platform designed to provide a secure, reliable and reproducible foundation for GPU-accelerated artificial intelligence workloads in an enterprise environment.

The platform runs on dedicated bare-metal infrastructure and uses Talos Linux as the immutable operating system for its Kubernetes control-plane and worker nodes. Cluster lifecycle management is implemented through Omni, providing a centralized and API-driven approach to provisioning, configuration, upgrades and secure access.

GPU worker nodes equipped with NVIDIA accelerators provide the compute capacity required for machine learning and inference workloads. The NVIDIA GPU Operator integrates the hardware with Kubernetes and manages the required device plugins, monitoring components and workload interfaces while respecting the immutable operating-system model of Talos Linux.

For workloads that process particularly sensitive data, the platform incorporates confidential-computing capabilities. Confidential Containers provide the workload integration required to use hardware-backed isolation and encrypted memory, reducing the trust placed in the host operating system and underlying infrastructure.

Dedicated GPU devices are assigned directly to confidential workloads through device passthrough. This enables hardware-accelerated processing while preserving workload isolation from other tenants and reducing exposure to the host environment.

A hardware security module protects cryptographic key material used by the platform and its confidential-computing workflows. This provides hardware-backed key protection for sensitive operations and helps ensure that cryptographic secrets are not exposed directly to the Kubernetes nodes or application workloads.

The network architecture is built around Cilium and the Kubernetes Gateway API. Dedicated internal and external gateways provide controlled access to platform services, while BGP is used to advertise service addresses directly into the surrounding network infrastructure. This avoids unnecessary proxy layers and enables Kubernetes services to participate directly in the data-center network.

Persistent storage is provided by Rook Ceph, using locally attached disks to create a distributed storage layer for block and file workloads. The storage platform is integrated with Kubernetes through CSI and monitored alongside the rest of the infrastructure.

All platform components are deployed declaratively through Argo CD. Applications, infrastructure services, security policies and configuration are maintained in Git repositories and continuously reconciled through GitOps workflows.

External Secrets integrates Kubernetes with Azure Key Vault so that sensitive values remain outside the Git repositories and are injected into workloads only when required. Certificate management and DNS automation provide repeatable and secure service exposure across internal and external environments.

Security controls are implemented at multiple layers. Kyverno validates Kubernetes resources and enforces platform policies, while Falco provides runtime threat detection. Workload requirements include restricted security contexts, controlled image registries, resource requests and limits, non-root execution and default network isolation.

A centralized observability platform based on Grafana, Mimir, Loki, Tempo and Alloy collects metrics, logs and traces from Kubernetes, Cilium, Ceph, GPU components, databases and application workloads. This provides a unified operational view of the platform while using Alloy and Mimir instead of operating a dedicated Prometheus server.

## Key Responsibilities

* Design and operation of a bare-metal Kubernetes platform based on Talos Linux
* Design of the physical and logical platform architecture
* Cluster lifecycle management and secure access through Omni
* Integration of GPU worker nodes and the NVIDIA GPU Operator
* Integration of confidential-computing capabilities for sensitive AI workloads
* Implementation of hardware-backed workload isolation and encrypted memory
* Integration of direct GPU passthrough into confidential workloads
* Integration of a hardware security module for protected cryptographic key handling
* Design of the Cilium-based network architecture, including BGP service advertisement and separate internal and external Gateway API exposure paths
* Deployment and operation of distributed storage using Rook Ceph
* Implementation of declarative platform delivery and lifecycle management using Argo CD and GitOps
* Integration of secrets, DNS and certificate automation using External Secrets, Azure Key Vault, ExternalDNS and cert-manager
* Implementation of Kubernetes admission policies with Kyverno and runtime threat detection with Falco
* Design and operation of the centralized observability platform based on Grafana, Mimir, Loki, Tempo and Alloy
* Creation and maintenance of Grafana dashboards and alerting rules
* Troubleshooting of networking, storage, GPU, confidential-computing and observability components
* Planning and execution of Kubernetes, Talos Linux and platform-component upgrades
* Documentation of platform architecture, bootstrap procedures and operational workflows

## Technologies & Methods

* Kubernetes
* Talos Linux
* Omni
* NVIDIA GPUs and NVIDIA GPU Operator
* Confidential Containers
* Confidential computing
* GPU passthrough
* Hardware security modules
* Hardware-backed workload isolation
* Cilium, eBPF and BGP
* Kubernetes Gateway API
* Rook Ceph
* Argo CD and GitOps
* Helm
* External Secrets and Azure Key Vault
* ExternalDNS and cert-manager
* Kyverno and Falco
* Grafana, Mimir, Loki, Tempo and Alloy
* OpenTelemetry
* CloudNativePG
* Bare-metal infrastructure
* Distributed systems
* Platform engineering

## Impact

* Established a reproducible Kubernetes platform for GPU-accelerated and confidential artificial intelligence workloads
* Introduced an immutable and API-driven operating model using Talos Linux and Omni
* Enabled Kubernetes-native access to dedicated NVIDIA GPU resources
* Enabled sensitive AI workloads to use hardware-backed isolation and encrypted memory
* Combined confidential workload execution with direct access to dedicated GPU acceleration
* Protected cryptographic key material through hardware-backed HSM integration
* Integrated Kubernetes services with the surrounding network through Cilium and BGP
* Established a Kubernetes-native distributed storage layer using locally attached worker-node disks
* Implemented declarative platform delivery and configuration through GitOps
* Centralized secrets management without storing sensitive values in Git repositories
* Introduced admission policy enforcement and runtime threat detection across the platform
* Provided centralized metrics, logs and traces for infrastructure and application workloads
* Created an extensible foundation for future AI services and workloads