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

Install AOB
===========

```
cat <<EOF | kubectl create -f -
---
apiVersion: v1
kind: Namespace
metadata:
  name: automation-broker-apb

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: automation-broker-apb
  namespace: automation-broker-apb

---
# Since the Broker APB will create CRDs and other privileged
# k8s objects, we need elevated permissions
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: automation-broker-apb
roleRef:
  name: cluster-admin
  kind: ClusterRole
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: automation-broker-apb
  namespace: automation-broker-apb

---
apiVersion: v1
kind: Pod
metadata:
  name: automation-broker-apb
  namespace: automation-broker-apb
spec:
  serviceAccount: automation-broker-apb
  containers:
    - name: apb
      image: docker.io/automationbroker/automation-broker-apb:latest
      args:
        - "provision"
        - "-e create_broker_namespace=true"
        - "-e broker_sandbox_role=admin"
        - "-e broker_dockerhub_tag=canary"
        - "-e broker_helm_enabled=true"
        - "-e broker_helm_url=https://kubernetes-charts.storage.googleapis.com"
        - "-e wait_for_broker=true"
      imagePullPolicy: IfNotPresent
  restartPolicy: Never
EOF
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
