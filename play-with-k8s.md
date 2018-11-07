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

Install helm
============
```
helm init
```

Get and install helm chart of the service-catalog
=================================================
```
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
helm install svc-cat/catalog --name catalog --namespace catalog
```

Play with dummy broker
======================
```
cd $GOPATH/src/github.com/kubernetes-incubator/service-catalog
helm install ./charts/ups-broker --name ups-broker --namespace ups-broker
kubectl create -f contrib/examples/walkthrough/ups-broker.yaml
```

and check

```
svcat get brokers
svcat describe broker ups-broker
kubectl get clusterservicebrokers ups-broker -o yaml
kubectl get clusterserviceclasses
svcat describe class user-provided-service
kubectl get clusterserviceclasses 4f6e6cf6-ffdd-425f-a2c7-3c9258ad2468 -o yaml
```

Install a serviceInstance and Binding

```
kubectl create namespace test-ns
kubectl create -f contrib/examples/walkthrough/ups-instance.yaml
svcat describe instance -n test-ns ups-instance
kubectl create -f contrib/examples/walkthrough/ups-binding.yaml
svcat describe binding -n test-ns ups-binding
```

Cleanup
=======
```
svcat unbind -n test-ns ups-instance
kubectl get secrets -n test-ns
svcat deprovision -n test-ns ups-instance
kubectl delete clusterservicebrokers ups-broker
svcat get classes
svcat get plans
helm delete --purge ups-broker
kubectl delete ns test-ns ups-broker
helm delete --purge catalog\nkubectl delete ns catalog
```
