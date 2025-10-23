# Distributed SQL Load Balancer

[![Go Version](https://img.shields.io/badge/Go-1.23-blue.svg)](https://golang.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A production-grade intelligent load balancer that distributes SQL queries across multiple database backends with automatic failure detection, health monitoring, and dynamic scaling capabilities.

## 🚀 Features

- **Intelligent Query Distribution** - Parses SQL queries and distributes them optimally across backends
- **Circuit Breaker Pattern** - Automatic failure detection with 3-state health monitoring (Healthy/Failed/Recovering)
- **Docker Auto-Discovery** - Zero-config backend registration using Docker API
- **Manual Scaling** - Add/remove backends on-demand with cooldown protection
- **Real-Time Monitoring** - 4 comprehensive dashboards with live metrics and visualizations
- **Response-Time Routing** - Selects backends based on health scores (error rate, latency, circuit state)
- **Metrics Preservation** - Maintains request counts and health data across discovery cycles
- **Zero-Downtime Scaling** - Seamless backend addition/removal without service interruption

## 📋 Table of Contents

- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [Dashboards](#dashboards)
- [API Endpoints](#api-endpoints)
- [Configuration](#configuration)
- [How It Works](#how-it-works)
- [Testing](#testing)
- [Development](#development)
- [Project Structure](#project-structure)
- [Contributing](#contributing)

## 🏗️ Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│   Load Balancer (Go)            │
│  - Query Parser                 │
│  - Circuit Breaker              │
│  - Health Monitor               │
│  - Docker Discovery             │
└────┬────────┬────────┬──────────┘
     │        │        │
     ▼        ▼        ▼
