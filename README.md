# Installation of weave flux

## Use cluster admin account to set helm
```
export USER=$(gcloud config get-value account)
kubectl create clusterrolebinding cluster-admin-binding-$(whoami) --clusterrole=cluster-admin --user=$USER
```

## Install HELM
```
kubectl create serviceaccount tiller --namespace kube-system
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account=tiller
helm update
```

## Install Weave Helm Repo
```
helm repo add weaveworks https://weaveworks.github.io/flux
kubectl apply -f https://raw.githubusercontent.com/weaveworks/flux/master/deploy-helm/flux-helm-release-crd.yaml
```

## Ensure Pod Security
```
kubectl apply -f flux/namespace.yaml
kubectl apply -f flux/psp.yaml
```

## Deploy Weave Flux 
```
export USER=engineer
export EMAIL=engineer@softserveinc.com
export REPO="git@github.com:runonautomation/flux-gitops.git" 

helm upgrade -i flux \
--set helmOperator.create=true \
--set helmOperator.createCRD=false \
--set git.url=$REPO \
--set git.user=$USER \
--set git.email=$EMAIL \
--set git.branch=demo1 \
--namespace flux \
weaveworks/flux
```

## Register SSH Key in cloud console
Get identity
```
fluxctl identity --k8s-fwd-ns flux
```

Enable the flux pod reading from the remote registry (add winpty for windows)
```
kubectl -n flux exec -it $(kubectl -n flux get po | grep flux | grep -v helm | grep -v memcache | awk '{print $1}') sh
```
Run 
```
ssh-keyscan -p 2022 REPO_HERE > ~/.ssh/known_hosts
```

# Helpers
#### Manually trigger the sync 
```
fluxctl --k8s-fwd-ns=flux sync
```