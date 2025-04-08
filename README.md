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
