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
minikube start --memory 4056 --kubernetes-version v1.11.0 --vm-driver xhyve

or 

minikube config set vm-driver hyperkit
minikube config set WantReportError true
minikube config set cpus 4
minikube config set kubernetes-version v1.11.0
minikube config set memory 5000
minikube start
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

Install OAB
===========

```
kubectl apply -f https://raw.githubusercontent.com/cmoulliard/cloud-native/master/oab/install.yml
```

**REMARK** : OAB can also be configured to contain the Helm's charts imported from `https://kubernetes-charts.storage.googleapis.com`. Then, install it using this command
`kubectl apply -f https://raw.githubusercontent.com/cmoulliard/cloud-native/master/oab/install-helm.yml`

Cleanup
=======

```bash
kubectl delete namespace/automation-broker-apb
kubectl delete clusterrolebinding.rbac.authorization.k8s.io/automation-broker-apb
```