# AWS FluxCD GitOps Repository

This repository contains the GitOps configuration for deploying and managing Kubernetes infrastructure, applications, and monitoring components using FluxCD on AWS. The repository follows a multi-cluster GitOps approach with different layers for infrastructure, configuration, and applications.

## 📁 Project Structure

```
aws-fluxcd/
├── apps/                       # Application definitions
│   └── base/                   # Base application configurations
│       └── microservices/      # Microservices applications
│           └── api/            # API service definitions
├── clusters/                   # Cluster-level FluxCD configurations
│   ├── apps.yaml              # Application deployments
│   ├── configs.yaml           # Infrastructure configurations
│   ├── infrastructure.yaml    # Infrastructure component definitions
│   ├── monitoring.yaml        # Monitoring stack definitions
│   └── flux-system/           # FluxCD system components
│       ├── gotk-components.yaml
│       ├── gotk-sync.yaml
│       └── kustomization.yaml
├── infrastructure/            # Infrastructure definitions
│   ├── configs/              # Configuration resources
│   │   └── base/             # Base configuration definitions
│   │       ├── cert-manager/
│   │       ├── certificates/
│   │       ├── cluster-autosacler/
│   │       ├── external-secret/
│   │       └── istio/
│   └── controllers/          # Infrastructure controllers
│       └── base/             # Base controller definitions
│           ├── ccm/          # Cloud Controller Manager
│           ├── cert-manager/ # Certificate management
│           ├── cluster-autoscaler/ # Cluster autoscaler
│           ├── external-dns/ # External DNS controller
│           ├── external-secrets/ # External secrets controller
│           ├── gateway-crds/ # Gateway API CRDs
│           ├── istio/        # Istio service mesh
│           ├── kured/        # Kubernetes reboot daemon
│           ├── longhorn/     # Longhorn storage
│           ├── metrics-server/ # Metrics server
│           ├── reloader/     # ConfigMap/Secret reloader
│           ├── spegel/       # Container registry mirror
│           └── velero/       # Backup and restore
└── monitoring/               # Monitoring stack definitions
    ├── configs/             # Monitoring configurations
    │   └── base/            # Base monitoring configs
    │       └── grafana/     # Grafana configuration
    └── controllers/         # Monitoring controllers
        └── base/            # Base monitoring controller definitions
            ├── kube-prometheus-stack/ # Prometheus monitoring stack
            └── loki/        # Loki log aggregation
```

## 🚀 Overview

This GitOps repository is organized into three main layers:

### 1. Infrastructure Layer (`infrastructure/`)
Contains all the infrastructure components required to run applications on Kubernetes, including:
- **CRDs**: Gateway API and other custom resource definitions
- **Storage**: Longhorn for persistent storage
- **Networking**: Istio service mesh, External DNS
- **Security**: Cert Manager, External Secrets
- **Observability**: Metrics server, monitoring components
- **Operations**: Cluster autoscaler, Kured, Velero backups

### 2. Configuration Layer (`infrastructure/configs/`)
Holds configuration resources that are applied to the cluster:
- Certificate configurations
- Istio configurations
- External secret definitions
- Cert Manager configurations

### 3. Application Layer (`apps/`)
Contains application deployments:
- **Microservices**: Application definitions for microservices architecture
- **API services**: Backend API service definitions

### 4. Monitoring Layer (`monitoring/`)
Comprehensive monitoring stack:
- **Prometheus**: Metrics collection with kube-prometheus-stack
- **Grafana**: Visualization and dashboards
- **Loki**: Log aggregation and storage

## 📋 Cluster Deployments

The `clusters/` directory contains FluxCD Kustomization resources that define what gets deployed to the cluster:

### `apps.yaml`
- Deploys microservices applications to the cluster
- Defines health checks for application deployments
- Sets up dependencies between services

### `configs.yaml`
- Manages infrastructure configurations
- Configures external secrets
- Sets up certificate management
- Configures Istio service mesh

### `infrastructure.yaml`
- Organized in phases for proper deployment order:
  - **Phase 1**: CRDs (Gateway API)
  - **Phase 2**: Basic infrastructure (metrics server, storage, registry mirror)
  - **Phase 3**: Core services (Istio, External Secrets, Cert Manager, etc.)

### `monitoring.yaml`
- Sets up the complete monitoring stack
- Deploys Prometheus with kube-prometheus-stack
- Configures Grafana with dashboards
- Installs Loki for log aggregation

## 🔧 Infrastructure Components

### Storage & Networking
- **Longhorn**: Distributed storage solution for Kubernetes
- **Istio**: Service mesh for traffic management, security, and observability
- **External DNS**: Automatic DNS record management

### Security & Certificates
- **Cert Manager**: Automatic certificate management
- **External Secrets**: Integration with external secret stores
- **CCM (Cloud Controller Manager)**: AWS-specific cloud integration

### Operations & Monitoring
- **Cluster Autoscaler**: Automatic node scaling
- **Metrics Server**: Resource metrics for HPA
- **Kured**: Automatic node rebooting for security updates
- **Velero**: Backup and disaster recovery
- **Reloader**: Automatic deployment restarts on ConfigMap/Secret changes

### Additional Tools
- **Spegel**: Container registry mirror for improved performance
- **External DNS**: Automatic DNS management for services

## 📊 Monitoring Stack

### Prometheus Ecosystem
- **Kube-Prometheus-Stack**: Complete monitoring solution with Prometheus, Alertmanager, and Grafana
- **Grafana**: Pre-configured dashboards for cluster and application monitoring
- **Loki**: Log aggregation system for Kubernetes workloads

## 🚀 Getting Started

### Prerequisites
- Kubernetes cluster
- FluxCD installed
- AWS credentials configured
- Git repository access

### Initial Setup
1. Bootstrap FluxCD in your cluster
2. Point FluxCD to this repository
3. Apply the cluster configurations:
   - Infrastructure (in phases)
   - Configurations
   - Monitoring
   - Applications

### Deployment Order
The infrastructure components are designed to be deployed in phases:
1. **CRDs** - Custom Resource Definitions
2. **Basic Infrastructure** - Core infrastructure components
3. **Core Services** - Service mesh, secrets, certificates
4. **Monitoring** - Complete monitoring stack
5. **Applications** - Business applications

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test your changes in a development environment
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⚠️ Notes

- Always verify changes in a development environment before applying to production
- Monitor cluster resources when adding new components
- Keep FluxCD components updated for security
- Regular backup schedules should be in place