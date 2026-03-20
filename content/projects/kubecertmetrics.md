---
title: "kubecertmetrics"
description: "Lightweight Prometheus exporter for monitoring TLS certificate expiration in Kubernetes and CI/CD environments."
tags: ["golang", "kubernetes", "prometheus", "observability", "devops", "security", "helm"]
codeberg: "https://codeberg.org/0hlov3/kubecertmetrics"
status: "active"
starts: "2025"
ends: "present"
---

> Lightweight Prometheus exporter and CLI tool for monitoring TLS certificate expiration across Kubernetes workloads and infrastructure.

Designed and implemented a production-ready monitoring tool to detect and alert on expiring TLS certificates. The tool can be used as a standalone CLI for CI/CD validation or as a long-running metrics endpoint integrated with Prometheus.

The project focuses on simplicity, reliability, and seamless integration into Kubernetes environments.

## Key Features

- CLI-based certificate validation with exit codes for CI/CD pipelines
- Prometheus metrics exporter for continuous monitoring
- Configurable thresholds for warning and critical states
- Flexible configuration via CLI flags, environment variables, and config files
- Graceful shutdown and scheduled checks

## Technologies & Methods

- Go (Cobra, Viper)
- Prometheus client libraries
- Kubernetes (Helm chart for deployment)
- Containerization (distroless images)
- CI/CD (Woodpecker, GoReleaser)
- GitOps-compatible deployment patterns

## Impact

- Enables proactive monitoring of TLS certificate expiration
- Prevents outages caused by expired certificates
- Easily integrates into Kubernetes clusters and CI/CD pipelines
- Demonstrates end-to-end ownership from development to deployment