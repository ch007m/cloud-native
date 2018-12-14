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
minikube start --memory 5000 --kubernetes-version v1.13.0 --vm-driver xhyve

or 

minikube config set vm-driver hyperkit
minikube config set WantReportError true
minikube config set cpus 4
minikube config set kubernetes-version v1.13.0
minikube config set memory 5000
minikube start
```

Check 
=====

Check that kubectl is properly configured by getting the cluster state:
```bash
kubectl cluster-info
Kubernetes master is running at https://192.168.65.37:8443
KubeDNS is running at https://192.168.65.37:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

See this page's doc for more info : https://kubernetes.io/docs/tasks/tools/install-kubectl/#check-the-kubectl-configuration

Use OpenShift console
=====================

```bash
cd /Users/dabou/Code/go-workspace/src/github.com/openshift/console
./build.sh 

export KUBECONFIG=$HOME/.kube/config
source ./contrib/environment.sh
./bin/bridge
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

Create Project and multiple contexts
====================================
```bash
kubectl create namespace my-spring-app
kubectl create namespace component-operator

# kubectl config set-cluster minikube --server=https://192.168.65.38:8443

see: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

kubectl config set-context component-operator --cluster=minikube --namespace=component-operator --user=minikube
kubectl config set-context my-spring-app --cluster=minikube --namespace=my-spring-app --user=minikube
```

Install Operator
================
```bash
kubectl config use-context component-operator
export operator_project=$GOPATH/src/github.com/snowdrop/component-operator

kubectl create -f $operator_project/deploy/sa.yaml
kubectl create -f $operator_project/deploy/cluster-rbac.yaml
kubectl create -f $operator_project/deploy/crd.yaml
kubectl create -f $operator_project/deploy/operator.yaml
```

Play with Component CRD
=======================
```bash
kubectl config use-context my-spring-app
export demo=/Users/dabou/Code/snowdrop/component-operator-demo
kubectl apply -f $demo/fruit-backend-sb/target/classes/META-INF/ap4k/component.yml

```

Cleanup
=======

```bash
kubectl delete namespace/automation-broker-apb
kubectl delete clusterrolebinding.rbac.authorization.k8s.io/automation-broker-apb
```