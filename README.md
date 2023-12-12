# argocd-deploy

This repository is meant to bootstrap Argo CD. Once setup, Argo CD will be able to manage itself. To do that, an 
[app-of-apps](https://github.com/blakepettersson/argocd-appsets) (or in reality an app-of-appsets) is configured in 
the `values.yaml` file, which in turn has an `ApplicationSet` pointing to this repo, along with another 
`ApplicationSet` which configures Prometheus and Grafana.

This is a Helm umbrella chart which bundles the [argo-cd](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd) 
chart, as well as the [argocd-apps](https://github.com/argoproj/argo-helm/tree/main/charts/argocd-apps) chart, which 
configures the app-of-appsets.

For Argo CD, the values used are the defaults coming from argo-helm, with a few exceptions: 
* The [resource tracking](https://argo-cd.readthedocs.io/en/stable/user-guide/resource_tracking/) has been set to `annotation+label`.
* Metrics and service monitors have been enabled in order for Argo CD metrics to be able to be scraped by Prometheus.
* The `argocd-server` service uses `NodePort.`
* TLS is disabled for `argocd-server`.

This setup currently assumes that this is meant to be run on a local minikube / Docker Desktop (hence the disabling of 
TLS and setting up of `NodePort`), but it shouldn't be too hard to extend this to work in other environments.

The initial setup is a one-liner, which will get the initial Argo CD up and running.

```shell
(kubectl get ns argocd || kubectl create ns argocd) && \ 
  helm template --namespace argocd --name-template=argocd . | kubectl apply -f -
```

Once that has been setup, you will need to configure the repository credentials in order for Argo CD to be able to manage 
itself in the future, as well as setting up of Prometheus and Grafana. You will first need to get a Github PAT through
some out-of-band means. Once you have gotten a PAT, you can then either set it up in the UI by logging in to 
http://localhost:30080, head to `Settings/Repositories` and add a repo that way, or you can add it with the following 
commands below:

```shell
argocd login localhost:30080 --insecure    
argocd repocreds add https://github.com/blakepettersson --username anything --password github_pat_<something>
```

In order for [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) 
to be deployed, we need to add a cluster label to our local cluster. Once the label has been set, the 
applicationset-controller will pick it up and then generate an app in the local cluster. In order to do that
we need to either configure it in the UI by logging in to http://localhost:30080, then to `Settings/Clusters` and edit
the cluster and adding a label with the key `environment` and the value `dev`.

As far as I know, there's currently no way to set labels on the `in-cluster` cluster with the `argocd` CLI. Once the 
`in-cluster` has been edited, you can do subsequent edits with `kubectl edit secret cluster-<clustername>` (since what
happens with the first edit is that a cluster secret gets created with all the attributes of the pre-existing 
`in-cluster`, but that does not initially exist).