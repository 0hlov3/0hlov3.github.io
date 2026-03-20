---
title: "EntraWatch"
description: "Internal cloud-native observability and governance tool for Azure Entra ID, providing visibility into identity hygiene, credential expiry, and tenant compliance."
tags: ["golang", "azure", "entra-id", "prometheus", "kubernetes", "helm", "internal-tooling", "security"]
status: "active"
starts: "2024"
ends: "present"
---

> Internal observability and governance tool for Azure Entra ID, built to monitor identity hygiene, credential expiry, and tenant-level compliance signals.

Design and development of **EntraWatch**, an internal cloud-native tool for monitoring and analyzing identity-related data in Microsoft Azure Entra ID. The application integrates with the Microsoft Graph API to collect information about users, service principals, and app registrations, exposing operational and governance-related insights through Prometheus-compatible metrics and HTTP endpoints.

The tool was designed to improve visibility into tenant health and identity hygiene by detecting missing user metadata, orphaned accounts, missing manager assignments, and expiring credentials. In addition to metrics, EntraWatch provides authenticated REST endpoints for querying identity-related information and supports automated notification workflows for follow-up actions.

EntraWatch is packaged as a containerized Go application and deployed to Kubernetes using Helm. It includes production-oriented operational features such as health, readiness, and liveness endpoints, Prometheus scraping, alerting rules, and hardened container security settings. The tool is integrated into Azure-based delivery workflows using Azure DevOps and Azure Pipelines.

## Key Responsibilities

- Design and development of an internal identity observability and governance tool in Go
- Integration with Microsoft Graph API for collecting Azure Entra ID user and application data
- Implementation of Prometheus-compatible metrics for tenant-level governance and security checks
- Development of authenticated REST endpoints for querying users, service principals, and app registrations
- Detection and monitoring of expiring client secrets and missing identity metadata
- Implementation of notification workflows for manager-based follow-up processes
- Packaging and deployment of the application to Kubernetes using Helm
- Integration of CI/CD workflows using Azure DevOps and Azure Pipelines
- Implementation of hardened container security defaults for production operation
- Integration into Prometheus-based monitoring and alerting workflows

## Technologies & Methods

- Go
- Azure Entra ID
- Microsoft Graph API
- Prometheus
- Kubernetes
- Helm
- Azure DevOps
- Azure Pipelines
- Azure Container Registry (ACR)
- REST APIs
- Identity Governance
- Observability and Alerting
- Secure container design

## Impact

- Increased visibility into identity hygiene and governance across Azure tenants
- Reduced operational risk by detecting expiring credentials before service disruption
- Enabled proactive monitoring of incomplete or inconsistent user directory data
- Integrated identity governance signals into existing Kubernetes-based observability workflows
- Provided an internal platform component for security, operations, and compliance-related monitoring