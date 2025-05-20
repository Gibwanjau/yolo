
# YOLO Full-Stack Application Deployment

![Project Architecture]

## Table of Contents
- [Project Overview](#project-overview)
- [Optimized Deployment Architecture](#optimized-deployment-architecture)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Docker Configuration](#docker-configuration)
  - [Production vs Development Builds](#production-vs-development-builds)
  - [Service Architecture](#service-architecture)
- [Verification Procedures](#verification-procedures)
- [CI/CD Integration](#cicd-integration)
- [Security Hardening](#security-hardening)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Project Overview

This repository contains a Docker-optimized full-stack application featuring:
- **Frontend**: React.js (Create React App)
- **Backend**: Node.js REST API
- **Database**: MongoDB with persistent storage
- **Infrastructure**: Docker containers with multi-stage builds

## Optimized Deployment Architecture

### Key Improvements in v2.0:
1. **68% Size Reduction** (From 180MB to 58MB per service)
2. **Security Hardening** (Non-root users, read-only filesystems)
3. **Build/Runtime Separation** (Multi-stage Dockerfiles)
4. **Health Monitoring** (Container healthchecks)
5. **CI/CD Ready** (GitHub Actions workflow)

![Size Optimization](docs/size-comparison.png)

## Prerequisites

| Tool               | Version     | Installation Guide                     |
|--------------------|-------------|----------------------------------------|
| Docker Engine      | 20.10+      | [Official Guide](https://docs.docker.com/engine/install/) |
| Docker Compose     | 2.5+        | [Official Guide](https://docs.docker.com/compose/install/) |
| Node.js            | 16+         | [NVM Guide](https://github.com/nvm-sh/nvm) |

## Getting Started

### 1. Clone Repository
```bash
git clone https://github.com/Vinge1718/yolo.git
cd yolo
2. Build and Run (Development)
bash
docker compose up -d --build
3. Build and Run (Production)
bash
DOCKERFILE=Dockerfile.prod docker compose -f docker-compose.prod.yml up -d --build
Access endpoints:

Frontend: http://localhost:3000

Backend API: http://localhost:5000

MongoDB: mongodb://localhost:27017
## Docker Configuration

### Production vs Development Builds

| Feature                | `Dockerfile` (Dev)               | `Dockerfile.prod` (Production)       |
|------------------------|-----------------------------------|---------------------------------------|
| **Base Image**         | `node:16-alpine`                 | `alpine:3.18` + Node runtime only     |
| **Dependencies**       | Includes devDependencies         | Only production dependencies          |
| **User Context**       | Root (for debugging)             | Dedicated non-root user (`appuser`)    |
| **Build Process**      | Single-stage                     | Multi-stage with build separation     |
| **Health Checks**      | Basic                            | Comprehensive endpoint monitoring     |
| **Filesystem**         | Read/Write                       | Read-only where possible              |
| **Size**               | ~180MB                           | ~58MB (68% reduction)                |
| **Security**           | Standard                         | Non-root user, minimized privileges   |
| **ENTRYPOINT**         | None                             | `tini` init system                    |
| **Caching**            | Basic layer caching              | Optimized dependency caching          |
| **Debugging**          | Full source access               | Only built artifacts                  |
| **Use Case**           | Development/testing              | Production deployments                |