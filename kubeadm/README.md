# Setup a kubeadm cluster

## Prerequisite

* Ubuntu 16.04+

## Install a specific version of kubadm on each node
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -q
sudo apt-get install -qy kubelet=<version> kubectl=<version> kubeadm=<version>
```

## Show existing version for kubeadm
```bash
curl -s https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages | grep -A 2 kubeadm | grep Version | awk '{print $2}'
```

## Init the kubeadm cluster
```bash
kubeadm init --kubernetes-version $(kubeadmversion -o short) --pod-network-cidr=192.168.0.0/16`
```

then paste the content that looks like 

```
kubeadm join 172.17.0.26:6443 --token xxxxx.yyyyyyyyyyyyyyyy --discovery-token-ca-cert-hash sha256:djsfsdlfkjlsdjkljlsdjfljsldjfljsdlfj
```

## Copy the config localy

```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

## Take a network policy 

https://kubernetes.io/docs/concepts/cluster-administration/addons/

### Calico 

https://docs.projectcalico.org/v3.3/getting-started/kubernetes/

1. Install an etcd instance with the following command.

```
kubectl apply -f \
https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/etcd.yaml
```

You should see the following output.

```
daemonset "calico-etcd" created
service "calico-etcd" created
```

2. Install the RBAC roles required for Calico

```
kubectl apply -f \
https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/rbac.yaml
```

You should see the following output.

```
clusterrole.rbac.authorization.k8s.io "calico-kube-controllers" created
clusterrolebinding.rbac.authorization.k8s.io "calico-kube-controllers" created
clusterrole.rbac.authorization.k8s.io "calico-node" created
clusterrolebinding.rbac.authorization.k8s.io "calico-node" created
```

3. Install Calico with the following command.

```
kubectl apply -f \
https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/calico.yaml
Note: You can also view the YAML in a new tab.
```

You should see the following output.

```
configmap "calico-config" created
secret "calico-etcd-secrets" created
daemonset.extensions "calico-node" created
serviceaccount "calico-node" created
deployment.extensions "calico-kube-controllers" created
serviceaccount "calico-kube-controllers" created
```

4. Confirm that all of the pods are running with the following command.

```bash
watch kubectl get pods --all-namespaces
```

Wait until each pod has the STATUS of Running.

```
NAMESPACE    NAME                                       READY  STATUS   RESTARTS  AGE
kube-system  calico-etcd-x2482                          1/1    Running  0         2m
kube-system  calico-kube-controllers-6ff88bf6d4-tgtzb   1/1    Running  0         2m
kube-system  calico-node-24h85                          2/2    Running  0         2m
kube-system  etcd-jbaker-virtualbox                     1/1    Running  0         6m
kube-system  kube-apiserver-jbaker-virtualbox           1/1    Running  0         6m
kube-system  kube-controller-manager-jbaker-virtualbox  1/1    Running  0         6m
kube-system  kube-dns-545bc4bfd4-67qqp                  3/3    Running  0         5m
kube-system  kube-proxy-8fzp2                           1/1    Running  0         5m
kube-system  kube-scheduler-jbaker-virtualbox           1/1    Running  0         5m
Press CTRL+C to exit watch.
```

5. Remove the taints on the master so that you can schedule pods on it.

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

It should return the following.

```
node "<your-hostname>" untainted
```

6. Confirm that you now have a node in your cluster with the following command.

```bash
kubectl get nodes -o wide
```

It should return something like the following.

```
NAME             STATUS  ROLES   AGE  VERSION  EXTERNAL-IP  OS-IMAGE            KERNEL-VERSION     CONTAINER-RUNTIME
<your-hostname>  Ready   master  1h   v1.8.x   <none>       Ubuntu 16.04.3 LTS  4.10.0-28-generic  docker://1.12.6
```

Congratulations! You now have a single-host Kubernetes cluster equipped with Calico.