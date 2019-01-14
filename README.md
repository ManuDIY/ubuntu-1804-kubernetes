# On-premis Kubernetes on Ubuntu 18.04
## Install Docker

```shell
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce
```

## Docker Proxy
```shell
sudo mkdir /etc/systemd/system/docker.service.d
sudo vi /etc/systemd/system/docker.service.d/http-proxy.conf
```
```
[Service]
Environment="HTTP_PROXY=https://web-proxy.corp.xxxxxx.com:8080/"
Environment="HTTPS_PROXY=https://web-proxy.corp.xxxxxx.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,localaddress,.localdomain.com"
```
```shell
systemctl daemon-reload
systemctl restart docker
```

## Install Kubernetes

```shell
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install kubeadm 
sudo swapoff -a
```
```shell
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### Install CNI Network Layer

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

### Join Kubernetes Nodes to Cluster ###

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.54.18:6443 --token 01i2nx.veva1984qksr731c --discovery-token-ca-cert-hash sha256:8773dc61ef0fc5bddc9bd231406c19241fac298f3c423d01741b12d17d5da23a

### Install MetalLB Plugin ###

kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/example-layer2-config.yaml

#### Configure MetalLB
```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.57.224/27

