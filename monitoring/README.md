# Monitoring Stack

This directory contains the Kubernetes manifests for deploying a complete monitoring stack using:

- **Prometheus**: For metrics collection and storage
- **Grafana**: For visualization and dashboards
- **Loki**: For log aggregation and querying
- **Alloy**: For log collection (Grafana's next-gen agent)

## Components

### Namespace
`namespace.yaml` - Creates the dedicated monitoring namespace.

### Prometheus
`prometheus-cluster-monitoring.yaml` - Deploys Prometheus with appropriate RBAC permissions and configurations for cluster-wide monitoring. It includes:
- ClusterRole and ClusterRoleBinding for accessing metrics
- ServiceAccount for Prometheus
- ConfigMap with Prometheus configuration
- Deployment for the Prometheus server
- Service to expose Prometheus

### Grafana
`grafana.yaml` - Deploys Grafana with:
- Deployment configuration
- ConfigMap for datasource configuration (connecting to Prometheus)
- Service to expose the Grafana UI

### Loki Stack with Alloy
`loki-cluster-logging.yaml` - Deploys the Loki stack and Alloy agent using Argo CD, including:
- Loki for log storage and querying
- Grafana Alloy for log collection from all pods/containers
- Configured to store logs by namespace, pod, container, and application

### Kubernetes Dashboard
`dashboard.yaml` - Deploys the Kubernetes Dashboard with:
- Admin user access
- ClusterRoleBinding for admin permissions
- Metrics scraper enabled
- NodePort service for access

## Setup Instructions

### Prerequisites
- Kubernetes cluster running
- Argo CD installed and configured
- `kubectl` CLI configured to access your cluster

### Installation

1. Apply the namespace first:
   ```bash
   kubectl apply -f namespace.yaml
   ```

2. Apply all monitoring components:
   ```bash
   kubectl apply -f prometheus-cluster-monitoring.yaml
   kubectl apply -f grafana.yaml
   kubectl apply -f loki-cluster-logging.yaml
   kubectl apply -f dashboard.yaml
   ```

Or let Argo CD handle the deployment by adding these manifests to your Argo CD application.

### Accessing the UIs

- **Grafana**: Access via NodePort service at `http://<node-ip>:30000` (adjust port as configured)
- **Prometheus**: Available at `http://prometheus.monitoring.svc.cluster.local:9090` within the cluster
- **Kubernetes Dashboard**: Access via NodePort service at `http://<node-ip>:30080`

### Default Credentials

- **Grafana**:
  - Username: admin
  - Password: admin (change on first login)
  
- **Kubernetes Dashboard**:
  - Use the token from the admin-user service account:
  ```bash
  kubectl -n kubernetes-dashboard create token admin-user
  ```

## Connecting Loki to Grafana

Once both Loki and Grafana are running, add Loki as a data source in Grafana:

1. Navigate to Configuration > Data Sources
2. Click "Add data source"
3. Select "Loki"
4. Set the URL to: `http://loki-stack:3100`
5. Click "Save & Test"

## About Grafana Alloy

Grafana Alloy is the next-generation observability agent that replaces Promtail and other single-purpose agents. It provides:

- **Unified Collection**: Collects logs, metrics, and traces with a single agent
- **Efficient Resources**: Lower resource footprint than running multiple agents
- **Advanced Processing**: Allows for filtering and transformation of logs before sending to Loki
- **Dynamic Configuration**: Supports dynamic configuration and service discovery

## Common Queries

### Loki Log Queries
- View all logs for a namespace: `{namespace="default"}`
- View all logs for a particular pod: `{pod="my-pod-name"}`
- Search for error messages: `{namespace="default"} |= "error"`
- View logs for a specific application: `{app="my-application"}` 