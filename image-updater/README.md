Apply the above manifest using:

```sh
kubectl apply -f install.yaml

```
# In future we need to make this use git-ops approach:
[Read This](https://argocd-image-updater.readthedocs.io/en/stable/basics/update-methods/#git-write-back-method)

This will setup the argocd-image-updater manifest 

After applying it change the ArgoCD Application manifest using:
```sh
kubectl edit -n argocd creditsea-backend
```
and add these fields: 

```
argocd-image-updater.argoproj.io/image-list: creditsea-backend = 533267250939.dkr.ecr.ap-south-1.amazonaws.com/creditsea-app:latest
argocd-image-updater.argoproj.io/<image>.update-strategy: digest
```

`digest` because we are cool and no package versioning get the latest one. 
