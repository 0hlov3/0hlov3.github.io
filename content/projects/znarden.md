---
title: "znarden"
description: "Terminal-based CLI and TUI client for interacting with Znuny/OTRS ticket systems, designed for operational visibility and efficient ticket workflows."
tags: ["golang", "cli", "tui", "bubbletea", "devops", "tooling", "automation"]
codeberg: "https://codeberg.org/0hlov3/znarden"
status: "active"
starts: "2025"
ends: "present"
---

> Terminal-based CLI and interactive dashboard for working with Znuny/OTRS ticket systems.

Designed and implemented a lightweight operational tool for interacting with Znuny/OTRS environments through both a command-line interface and an interactive terminal dashboard built with Bubble Tea. The tool integrates with the Znuny Generic Ticket Connector REST API and supports multiple ticket systems through a flexible configuration model.

The project focuses on improving operational efficiency when working with ticket systems by providing fast access to queues, ticket details, and articles directly from the terminal. It supports both scripted usage via CLI commands and interactive workflows via a live-refreshing TUI dashboard.

The application is built as a production-ready Go project with automated testing, multi-platform release automation, and container image delivery.

## Key Features

- CLI commands for listing, retrieving, and showing tickets
- Interactive terminal dashboard (TUI) with live refresh
- Support for multiple Znuny/OTRS systems in a single configuration
- Configuration via TOML files and environment variables
- Secure configuration output with redacted credentials
- Automated multi-platform release pipeline with container image builds

## Technologies & Methods

- Go (Cobra, Viper)
- Bubble Tea / Lipgloss (terminal UI)
- Znuny / OTRS Generic Ticket Connector REST API
- REST-based integrations
- CI/CD (Woodpecker, GoReleaser)
- Containerization (distroless images)
- Multi-platform release automation

## Impact

- Improves operational visibility into ticket queues and ticket details
- Reduces friction when working with support and service desk workflows
- Provides both scripted and interactive workflows for different operational use cases
- Demonstrates end-to-end ownership from application design to packaging and release