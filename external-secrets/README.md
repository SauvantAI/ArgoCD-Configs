# External Secrets Operator

This directory contains the configuration for the External Secrets Operator, which synchronizes secrets from external API services into Kubernetes.

## Components

### Secret Operator

`secret-operator.yaml` - Deploys the External Secrets Operator using ArgoCD, including:
- The operator deployment
- CRDs for managing external secrets
- RBAC permissions
- Service accounts with appropriate AWS IAM role annotations

## How External Secrets Works

1. The External Secrets Operator connects to external secrets management systems (like AWS Secrets Manager, HashiCorp Vault, etc.)
2. It fetches secrets from these external sources
3. It creates corresponding Kubernetes Secret objects in your cluster

## Setup Instructions

### Prerequisites

- Kubernetes cluster running
- ArgoCD installed and configured
- Access to external secrets provider (e.g., AWS IAM role for accessing AWS Secrets Manager)

### Installation

1. Apply the operator configuration:
   ```bash
   kubectl apply -f secret-operator.yaml
   ```

Or let ArgoCD handle the deployment.

### Configuring Secret Stores

After installing the operator, you'll need to configure a `SecretStore` or `ClusterSecretStore` resource to define how to connect to your external secrets provider. Example in the `k8s/secret-store.yaml` file.

### Creating External Secrets

Once you have a `SecretStore` configured, you can create `ExternalSecret` resources to fetch specific secrets. Example in the `k8s/externalSecret.yaml` file.

## AWS Integration

The configuration in this directory is set up to work with AWS Secrets Manager. The service account has an annotation linking it to an AWS IAM role:

```yaml
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::533267250939:role/external-secrets-role
```

### Required AWS IAM Permissions

The IAM role used must have these permissions:
- `secretsmanager:GetSecretValue`
- `secretsmanager:DescribeSecret`
- `secretsmanager:ListSecretVersionIds`

### AWS IAM Role Trust Relationship

The role must have a trust relationship with the Kubernetes service account:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::533267250939:oidc-provider/oidc.eks.your-region.amazonaws.com/id/YOUR_OIDC_ID"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.your-region.amazonaws.com/id/YOUR_OIDC_ID:sub": "system:serviceaccount:external-secrets:external-secrets-operator"
        }
      }
    }
  ]
}
```

## Usage Example

1. Create a secret in AWS Secrets Manager
2. Define an ExternalSecret resource:
   ```yaml
   apiVersion: external-secrets.io/v1beta1
   kind: ExternalSecret
   metadata:
     name: my-secret
     namespace: my-namespace
   spec:
     refreshInterval: "15m"
     secretStoreRef:
       name: aws-secretstore
       kind: SecretStore
     target:
       name: my-k8s-secret
     data:
     - secretKey: password
       remoteRef:
         key: my-aws-secret
         property: password
     ```

3. The operator will create a Kubernetes Secret named `my-k8s-secret` with data from AWS Secrets Manager

## Troubleshooting

- Check the logs of the External Secrets Operator pods
- Verify IAM permissions and trust relationships
- Ensure the External Secret references the correct Secret Store
- Verify the remote secret exists in your external provider

## Additional Resources

- [External Secrets Operator Documentation](https://external-secrets.io/latest/)
- [AWS Secrets Manager Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [EKS IAM Roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) 