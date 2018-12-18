# Install k8s tools

```
brew update
brew cask install minikube
brew install kubernetes-cli
brew install kubernetes-service-catalog-client
brew install kubernetes-helm
```

## Start minikube

```
minikube start --memory 5000 --kubernetes-version v1.13.0 --vm-driver xhyve

or 

minikube config set vm-driver virtualbox
minikube config set WantReportError true
minikube config set cpus 4
minikube config set kubernetes-version v1.13.0
minikube config set memory 5000
minikube addons enable ingress
minikube start
```

## Create namespaces and kube's contexts

```bash
see: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
# Using minikube
# kubectl config set-cluster minikube --server=https://192.168.65.38:8443
kubectl create namespace my-spring-app
kubectl create namespace component-operator
kubectl config set-context component-operator --cluster=minikube --namespace=component-operator --user=minikube
kubectl config set-context my-spring-app --cluster=minikube --namespace=my-spring-app --user=minikube
```

## Issue with minikube 0.31.0

```bash
# See : https://github.com/kubernetes/minikube/issues/3455
# See : https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/
scp -i $(minikube ssh-key) ./minikube/bootstrap-kubelet.conf docker@$(minikube ip):/etc/kubernetes/bootstrap-kubelet.conf
```

## Start Using microk8s

```bash
export VM_NAME=microk8s
multipass launch --name $VM_NAME --cpus 4 --mem 4G --disk 40G
multipass exec $VM_NAME -- sudo snap install microk8s --classic --channel=1.13/stable
multipass exec $VM_NAME -- /snap/bin/microk8s.enable dns ingress storage
# My pods can’t reach the internet (but my MicroK8s host machine can).
multipass exec $VM_NAME -- sudo iptables -P FORWARD ACCEPT
multipass exec $VM_NAME -- sudo ufw default allow routed
# My dns and dashboard pods are CrashLooping.
multipass exec $VM_NAME -- sudo ufw allow in on cbr0 
multipass exec $VM_NAME -- sudo ufw allow out on cbr0
multipass exec $VM_NAME -- /snap/bin/microk8s.kubectl config view --raw > $HOME/.kube/config
export IP=$(multipass info $VM_NAME --format json  | jq -r .info.microk8s.ipv4[0])
sed -i'.bk' -e "s/127.0.0.1/$IP/g" $HOME/.kube/config
# TODO : Check if this command is needed -> multipass exec $VM_NAME -- /snap/bin/microk8s.kubectl proxy --accept-hosts=.* --address=0.0.0.0
```

## Create namespaces and kube's contexts

```bash
kubectl create namespace my-spring-app
kubectl create namespace component-operator
kubectl config set-context component-operator --cluster=microk8s-cluster --namespace=component-operator --user=admin
kubectl config set-context my-spring-app  --cluster=microk8s-cluster --namespace=my-spring-app --user=admin
```

## Install the PV (needed for microk8s)

```bash
kubectl apply -f microk8s/pv001.yml
kubectl apply -f microk8s/pv002.yml
kubectl apply -f microk8s/pv003.yml
kubectl apply -f microk8s/pv004.yml
kubectl apply -f microk8s/pv005.yml
```

# Verify if the cluster is working correctly  

Check that kubectl is properly configured by getting the cluster state:
```bash
kubectl cluster-info
Kubernetes master is running at https://192.168.65.37:8443
KubeDNS is running at https://192.168.65.37:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

See this page's doc for more info : https://kubernetes.io/docs/tasks/tools/install-kubectl/#check-the-kubectl-configuration

# Setup the OpenShift console

```bash
cd $GOPATH/src/github.com/openshift/console
./build.sh 

export KUBECONFIG=$HOME/.kube/config
source ./contrib/environment.sh
./bin/bridge
```

# Install helm's Tiller on k8s

```
helm init
wait till tiller's pod is running
```

# Get and install helm chart of the k8s service-catalog
```
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
helm install svc-cat/catalog --name catalog --namespace catalog
```

# Install OAB

```
curl https://raw.githubusercontent.com/openshift/ansible-service-broker/master/ansible_role/apb/install.yaml | kubectl create -f -
```

**WARNING** : Change within the `configMap`, the field `auto_escalate` to true otherwise, the Broker's pod will report this error `Unable to retrieve cluster roles rules from cluster`

**REMARK** : OAB can also be configured to contain the Helm's charts imported from `https://kubernetes-charts.storage.googleapis.com`. Then, install it using this command
`kubectl apply -f https://raw.githubusercontent.com/cmoulliard/cloud-native/master/oab/install-helm.yml`

# Install The Component Operator

```bash
kubectl config use-context component-operator
export operator_project=$GOPATH/src/github.com/snowdrop/component-operator

kubectl create -f $operator_project/deploy/sa.yaml
kubectl create -f $operator_project/deploy/cluster-rbac.yaml
kubectl create -f $operator_project/deploy/crd.yaml
kubectl create -f $operator_project/deploy/operator.yaml
```

# Play with Component CRD

```bash
kubectl config use-context my-spring-app
cd /Users/dabou/Code/snowdrop/component-operator-demo
kubectl apply -f fruit-backend-sb/target/classes/META-INF/ap4k/component.yml
kubectl apply -f fruit-client-sb/target/classes/META-INF/ap4k/component.yml
./k8s_push_start.sh fruit-backend sb
./k8s_push_start.sh fruit-client sb

curl  --resolve fruit-client-sb:80:192.168.65.42 -k http://fruit-client-sb/api/client 
curl  --resolve fruit-backend-sb:80:192.168.65.42 -k http://fruit-backend-sb/api/fruits 

cd /Users/dabou/MyProjects/cloud-native
```

# Cleanup

```bash
kubectl delete namespace/automation-broker-apb
kubectl delete clusterrolebinding.rbac.authorization.k8s.io/automation-broker-apb
```