Install local K8S for development with Minikube

```shell

#!/bin/bash
#Install pre-requisites
apt-get update && apt-get install virtualbox containerd -y
curl -fsSl https://get.docker.com | bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
mkdir -p /usr/local/bin/
install minikube /usr/local/bin/
cp minikube /usr/local/bin && rm minikube

#Install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl && mv kubectl /usr/local/bin/

#minikube start --vm-driver=none
sudo mv /root/.kube /root/.minikube $HOME
sudo chown -R $USER $HOME/.kube $HOME/.minikube
echo br_netfilter > /etc/modules-load.d/k8s.conf
echo ip_vs_rr >> /etc/modules-load.d/k8s.conf
echo ip_vs_wrr >> /etc/modules-load.d/k8s.conf
echo ip_vs_sh >> /etc/modules-load.d/k8s.conf
echo nf_conntrack_ipv4 >> /etc/modules-load.d/k8s.conf
echo ip_vs >> /etc/modules-load.d/k8s.conf

#Update distro and install kubelet kubeadm kubectl
apt-get update -y && apt-get upgrade -y && apt-get install apt-transport-https -y
modprobe br_netfilter ip_vs_rr ip_vs_wrr ip_vs_sh nf_conntrack_ipv4 ip_vs
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update -y && apt-get install -y kubelet kubeadm kubectl
docker info | grep -i cgroup
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

#Restart services
systemctl daemon-reload
systemctl restart kubelet

#Disable Swap
swapoff -a 

#Disable automatic initialization
sed 's/^/#/' /etc/fstab

#Download images of minikube
kubeadm config images pull

#Config certificate minikube
minikube config set embed-certs true

#Minikube start
kubeadm init --ignore-preflight-errors all

#First deploy of POD network Weave
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" 

#Completion commands of kubectl
echo "source <(kubectl completion bash)" >> ~/.bashrc

#Disable firewall in your repo (optional)
ufw disable && iptables -L

```
