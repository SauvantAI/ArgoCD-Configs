# Commands if anything fails 

### *Things to keep in mind:*
1. Its specified inside terraform code that there are 2 types of Nodes running: 
    1. General Purpose Group(t3.medium) : Does not autoscale and handled by Lord Voldemort used for general purpose thingy, Handles nginx-controller,cert-manager,monitoring
    2. Karpenter Group(m5-large): Ability to autoscale (yay) will be used for: kube-system, argocd, application
2. Some of the packages are installed using Helm some with kubectl and others with gitops (this is because : why not) #TODO: needs to be cleaned asap
3. 

Manually done stuff: Installing these manifests manually :(  -> This needs to be ported to gitops as mentioned above

```sh
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
```


```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```


```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/aws/deploy.yaml
```

## To add auto-image updater do : 

```sh
kubectl get -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

# Credit-Sea ArgoCD GitOps Configurations

This repository contains the Kubernetes configurations for deploying and managing infrastructure and applications using the GitOps methodology with ArgoCD.

## Repository Structure

- **argo-config/**: ArgoCD specific configurations
  - `image-updater.yaml`: Configuration for automatically updating container images
  - `README.md`: Documentation for ArgoCD setup

- **monitoring/**: Observability infrastructure
  - `namespace.yaml`: Creates monitoring namespace
  - `prometheus-cluster-monitoring.yaml`: Prometheus setup for metrics collection
  - `grafana.yaml`: Grafana setup for dashboards and visualization
  - `loki-cluster-logging.yaml`: Loki and Grafana Alloy for log aggregation and collection
  - `dashboard.yaml`: Kubernetes dashboard for cluster overview
  - `README.md`: Documentation for monitoring setup

- **k8s/**: Kubernetes resource templates
  - `deployment.yaml`: Application deployment configuration
  - `service.yaml`: Service configuration for exposing applications
  - `configmap.yaml`: ConfigMap for storing configuration data
  - `horizontal-pod-scaler.yaml`: Automatic scaling configuration
  - `certificate.yaml`: TLS certificate configuration
  - `cluster-issuer.yaml`: Certificate issuer configuration
  - `secret-store.yaml`: Secret store configuration
  - `externalSecret.yaml`: External secrets configuration
  - `README.md`: Documentation for Kubernetes resources

- **external-secrets/**: External secrets operator configuration
  - `secret-operator.yaml`: External Secrets Operator deployment
  - `README.md`: Documentation for external secrets setup

## GitOps Workflow

This repository follows the GitOps methodology:

1. All infrastructure and application configurations are defined as code in this repository
2. ArgoCD monitors this repository for changes
3. When changes are detected, ArgoCD automatically applies them to the Kubernetes cluster
4. The ArgoCD Image Updater automatically updates application versions when new container images are available

## Getting Started

### Prerequisites

- Kubernetes cluster
- ArgoCD installed and configured
- `kubectl` CLI tool
- Access to container registries

### Initial Setup

1. Deploy ArgoCD if not already installed:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. Deploy the ArgoCD Image Updater:
   ```bash
   kubectl apply -f argo-config/image-updater.yaml
   ```

3. Create the monitoring infrastructure:
   ```bash
   kubectl apply -f monitoring/namespace.yaml
   kubectl apply -f monitoring/
   ```

4. Set up the External Secrets Operator:
   ```bash
   kubectl apply -f external-secrets/secret-operator.yaml
   ```

5. Configure your application in ArgoCD to use the resources in the `k8s/` directory

### Accessing ArgoCD

1. Forward the ArgoCD API server port:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

2. Access the ArgoCD UI at https://localhost:8080

3. Log in with the initial admin credentials:
   ```bash
   # Get the initial admin password
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

## Monitoring Stack Access

1. Access Grafana:
   ```bash
   kubectl port-forward svc/grafana -n monitoring 3000:80
   ```
   Then open http://localhost:3000 in your browser

2. Access Prometheus:
   ```bash
   kubectl port-forward svc/prometheus -n monitoring 9090:9090
   ```
   Then open http://localhost:9090 in your browser

3. Access Kubernetes Dashboard:
   ```bash
   kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8443:443
   ```
   Then open https://localhost:8443 in your browser

## Contributing

1. Create a new branch for your changes
2. Make your changes to the configuration files
3. Submit a pull request
4. After review and approval, the changes will be merged to the main branch
5. ArgoCD will automatically apply the changes to the cluster

## Security Considerations

- Never commit sensitive data like passwords or tokens to this repository
- Use the External Secrets Operator to manage sensitive information
- Ensure proper RBAC permissions for all components
- Follow the principle of least privilege when defining roles

## Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [External Secrets Operator Documentation](https://external-secrets.io/latest/)
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Grafana Alloy Documentation](https://grafana.com/docs/alloy/latest/)
- [Keel](https://keel.sh/docs/)
