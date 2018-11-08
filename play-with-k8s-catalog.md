Install k8s tools
=================
```
brew update
brew cask install minikube
brew install kubernetes-cli
brew install kubernetes-service-catalog-client
brew install kubernetes-helm
```
Start minikube
==============
```
minikube start --memory 4056 --kubernetes-version v1.11.0 --vm-driver hyperkit | xhyve
```

Install helm's Tiller on k8s
============================
```
helm init
wait till tiller's pod is running
```

Get and install helm chart of the service-catalog
=================================================
```
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
helm install svc-cat/catalog --name catalog --namespace catalog
```

Install AOB
===========

```
kubectl apply -f https://raw.githubusercontent.com/cmoulliard/cloud-native/master/oab/install.yml
```
