# argocd-deploy

## To bootstrap

```shell
(kubectl get ns argocd || kubectl create ns argocd) && \ 
  helm template --namespace argocd --name-template=argocd . | kubectl apply -f -
argocd login localhost:30080 --insecure    
argocd repocreds add https://github.com/blakepettersson  --username anything --password github_pat_<something>
```