# AWS FluxCD GitOps Repository

This repository contains the GitOps configuration for deploying and managing a Kubernetes cluster on AWS using FluxCD. It's structured to manage infrastructure, applications, and monitoring components in a declarative way.

## 📁 Directory Structure

*   `apps/`: Contains the Kubernetes manifests for your applications.
    *   `base/microservices/api/`: A sample API microservice.
*   `clusters/`: Defines the FluxCD Kustomizations that deploy the entire stack. This is the entry point for FluxCD.
*   `infrastructure/`: Contains the core infrastructure components for the cluster.
    *   `configs/`: Configurations for the infrastructure components (e.g., certificates, gateways).
    *   `controllers/`: The infrastructure components themselves (e.g., Istio, Velero).
*   `monitoring/`: Contains the monitoring stack.
    *   `configs/`: Configurations for the monitoring components (e.g., Grafana dashboards).
    *   `controllers/`: The monitoring components themselves (e.g., Prometheus, Loki).

## 🚀 Core Infrastructure

The following components are deployed from the `infrastructure/controllers/base/` directory:

### Velero

*   **Purpose**: Velero is used for backing up and restoring your Kubernetes cluster resources and persistent volumes.
*   **Configuration (`values.yaml`)**:
    *   **Provider**: AWS (`provider: "aws"`)
    *   **Bucket**: `backup-velero-k8s`
    *   **Prefix**: `prod`
    *   **Backup Storage Location**: A default backup storage location is configured to use AWS S3.
    *   **Plugins**: The AWS plugin is enabled.
    *   **Credentials**: Velero is configured to use a Kubernetes secret named `velero-aws-creds` which is created by `external-secrets` from Vault.

### Longhorn

*   **Purpose**: Longhorn provides persistent storage for your stateful applications.
*   **Configuration (`values.yaml`)**:
    *   **Default Replica Count**: 2 (`defaultReplicaCount: 2`)
    *   **Storage Over Provisioning**: 100% (`storageOverProvisioningPercentage: 100`)
    *   **Node Down Pod Deletion Policy**: `delete-both-statefulset-and-deployment-pod`
    *   **Default Class**: Longhorn is configured as the default storage class.

### Istio

*   **Purpose**: Istio is a service mesh that provides traffic management, security, and observability for your microservices.
*   **Configuration (`values.yaml`)**:
    *   **Gateway API**: Enabled (`PILOT_ENABLE_GATEWAY_API: true`)
    *   **Tracing**: Enabled (`telemetry.v2.enabled: true`)
    *   **Ingress Gateway**: An Istio ingress gateway is configured as a LoadBalancer service.

### cert-manager

*   **Purpose**: cert-manager automates the management of TLS certificates.
*   **Configuration (`values.yaml`)**:
    *   **Gateway API**: Enabled (`enableGatewayAPI: true`)
    *   **ClusterIssuer**: A `ClusterIssuer` named `letsencrypt-production-wildcard` is configured to issue wildcard certificates for `sirpify.com` using Let's Encrypt and DigitalOcean for DNS-01 challenges.

### ExternalDNS

*   **Purpose**: ExternalDNS automatically manages DNS records for your services.
*   **Configuration (`values.yaml`)**:
    *   **Provider**: DigitalOcean (`provider: digitalocean`)
    *   **Domain Filter**: `sirpify.com`
    *   **Sources**: It watches Services, Ingresses, and Gateway HTTPRoutes.

### External Secrets

*   **Purpose**: This controller fetches secrets from external secret stores (in this case, Vault) and makes them available as Kubernetes Secrets.
*   **Configuration**:
    *   A `ClusterSecretStore` named `vault-backend` is configured to connect to a Vault instance at `https://vault-lab.sirpify.com`.

### Cluster Autoscaler

*   **Purpose**: Automatically adjusts the number of nodes in your cluster.
*   **Configuration**:
    *   **Cloud Provider**: AWS (`cloudProvider: aws`)
    *   **Auto Scaling Group**: `aws-k8s-cluster-worker-asg`
    *   **Min/Max Size**: 0 to 2 nodes.

### Kured

*   **Purpose**: Kured (Kubernetes Reboot Daemon) safely reboots nodes when required by the underlying OS.
*   **Configuration (`values.yaml`)**:
    *   **Schedule**: Runs every hour between 00:00 and 05:00 Asia/Kolkata time.

### Other Infrastructure Components

*   **`ccm`**: AWS Cloud Controller Manager, for integrating with AWS APIs.
*   **`gateway-crds`**: Installs the Kubernetes Gateway API CRDs.
*   **`metrics-server`**: Provides resource metrics for autoscaling.
*   **`reloader`**: Automatically restarts pods when their ConfigMaps or Secrets are updated.
*   **`spegel`**: A distributed container registry mirror to speed up image pulls.

## 📊 Monitoring

The monitoring stack is deployed from the `monitoring/controllers/base/` directory.

### Kube Prometheus Stack

*   **Purpose**: A full monitoring stack including Prometheus, Grafana, and Alertmanager.
*   **Configuration (`values.yaml`)**:
    *   **Prometheus**:
        *   **Storage**: Uses Longhorn for persistent storage with a 1Gi request.
        *   **Retention**: 1 day.
    *   **Grafana**:
        *   **Persistence**: Enabled with a 4Gi Longhorn volume.
        *   **Admin Credentials**: Managed by `external-secrets` from Vault.
        *   **Plugins**: `grafana-polystat-panel` and `redis-datasource` are installed.
        *   **Datasources**: A Loki datasource is pre-configured.
    *   **Alertmanager**: Enabled and configured to use an external secret for its configuration.

### Loki

*   **Purpose**: Loki is a log aggregation system.
*   **Configuration (`values.yaml`)**:
    *   **Deployment Mode**: `SingleBinary`
    *   **Storage**: Uses a 500m Longhorn volume.
*   **Alloy**: The `alloy` component is Grafana Alloy, which is used to collect and process logs before sending them to Loki.

## 📦 Applications

The `apps/` directory contains a sample microservice:

*   **`first-api`**: A simple API with a Deployment and Service. An `HTTPRoute` is also defined to expose it through the Istio gateway.

## ⚙️ FluxCD Deployment

The `clusters/` directory is the heart of the FluxCD deployment. The YAML files in this directory are `Kustomization` resources that tell FluxCD what to deploy and in what order.

*   **`infrastructure.yaml`**: Deploys the core infrastructure components in phases to ensure dependencies are met.
*   **`configs.yaml`**: Deploys the configurations for the infrastructure components.
*   **`monitoring.yaml`**: Deploys the monitoring stack.
*   **`apps.yaml`**: Deploys the sample application.

This structured approach allows for a clear separation of concerns and ensures that the cluster is always in the state defined in this repository.