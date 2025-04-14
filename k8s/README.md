# Kubernetes Configuration

This directory contains Kubernetes manifests used in the ArgoCD GitOps workflow for deploying applications to Kubernetes.

## Components

### Application Resources

- **deployment.yaml**: Defines the Kubernetes deployment for the application, including the container image, environment variables, resource limits, and probes.

- **service.yaml**: Exposes the application deployment as a service within the Kubernetes cluster.

- **configmap.yaml**: Stores non-sensitive configuration data that can be consumed by pods.

- **horizontal-pod-scaler.yaml**: Configures automatic scaling of pods based on CPU or memory utilization.

### Security Resources

- **externalSecret.yaml**: Defines external secrets to be fetched from an external secret management system like AWS Secrets Manager or HashiCorp Vault.

- **secret-store.yaml**: Configures the connection to external secret providers.

- **certificate.yaml**: Configures TLS certificates for secure HTTPS communication.

- **cluster-issuer.yaml**: Defines a cluster-wide certificate issuer for automatic certificate management.

### Infrastructure

- **kustomization.yaml**: Organizes and customizes multiple Kubernetes resources without modifying the original YAML files.

## Usage

### Prerequisites

- Kubernetes cluster
- ArgoCD installed
- External Secrets Operator configured (for secrets management)
- Cert-Manager installed (for certificate management)

### Deployment Process with ArgoCD

1. ArgoCD monitors this Git repository for changes
2. When changes are detected, ArgoCD applies the manifests to your Kubernetes cluster
3. Resources are created, updated, or deleted according to the desired state in Git

### Manually Applying Resources (for testing)

```bash
# Apply all resources
kubectl apply -k .

# Or apply individual resources
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
# etc.
```

## Best Practices

1. **Resource Requests and Limits**: Always specify resource requests and limits in your deployments to ensure proper scheduling and prevent resource starvation.

2. **Health Checks**: Include readiness and liveness probes in your deployments for better application health monitoring.

3. **Security**: Use external secrets for sensitive information and ensure proper RBAC controls.

4. **Scaling**: Configure horizontal pod autoscaling for applications with variable workloads.

5. **Networking**: Use services and ingresses appropriately for internal and external network exposure.

## Example Workflow

1. Make changes to Kubernetes manifests in this directory
2. Commit and push changes to Git repository
3. ArgoCD detects changes and automatically syncs with the cluster
4. Monitor the deployment in ArgoCD dashboard

## Troubleshooting

- **Deployment Issues**: Check ArgoCD dashboard for sync status and potential errors
- **Pod Issues**: Use `kubectl describe pod <pod-name>` to see detailed pod status
- **Secret Issues**: Verify external secret provider is accessible and configured correctly

For more detailed information, refer to the [ArgoCD documentation](https://argo-cd.readthedocs.io/en/stable/). 