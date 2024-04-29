# Install Kubernetes on Master Node 
## Step 1: Update and Upgrade Ubuntu (all nodes)

Upgrade apt packages
```shell
sudo apt-get update
sudo apt upgrade
```
Create a configuration file for containerd:
```shell
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

## Step 2: Disable Swap (all nodes)
```shell
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Step 3: Add Kernel Parameters (all nodes)
```shell
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

Configure the critical kernel parameters for Kubernetes using the following:

```shell
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
Apply new settings:
```shell
sudo sysctl --system
```
## Step 4: Install Containerd Runtime (all nodes)
We are using the containerd runtime. Install containerd and its dependencies with the following commands:
```shell
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

Install containerd:
```shell
sudo apt-get update && sudo apt-get install -y containerd
```
Enable the Docker repository:
```shell
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
Update the package list and install containerd:
```shell
sudo apt update
sudo apt install -y containerd.io
```
Configure containerd to start using systemd as cgroup:
```shell
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
Restart and enable the containerd service:

```shell
sudo systemctl restart containerd
sudo systemctl enable containerd
```
## Step 5: Add Apt Repository for Kubernetes (all nodes)

Kubernetes packages are not available in the default Ubuntu 22.04 repositories. Add the Kubernetes repositories with the following commands:
```shell
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"```
```
Install kub:
```shell

sudo apt update

sudo apt install -y kubeadm=1.28.9-2.1 kubelet=1.28.9-2.1 kubectl=1.28.9-2.1
sudo apt-mark hold kubelet kubeadm kubectl

```

## Step 7: Initialize Kubernetes Cluster with Kubeadm (master node)

```shell
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.28.9
```

Set kubectl access:

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


Test access to cluster:

```shell
kubectl get nodes
```

Install the Calico Network Add-On -

On the Control Plane Node, install Calico Networking:

```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml   
```


Wait for 2-4 Min and Check the status of the control plane node:

```shell
kubectl get nodes
```

Join the Worker Nodes to the Cluster

In the Control Plane Node, create the token and copy the kubeadm join command (NOTE: The join command can also be found in the output from kubeadm init command):

```shell
kubeadm token create --print-join-command
```
In both Worker Nodes, paste the kubeadm join command to join the cluster. Use sudo to run it as root:

sudo kubeadm join ...
In the Control Plane Node, view cluster status (Note: You may have to wait a few moments to allow all nodes to become ready):

```kubectl get nodes```

## Step 8: Add Worker Nodes to the Cluster (worker nodes)
```shell
kubeadm join 
```
## Step :9 Install Kubernetes Network Plugin (master node)
```shell
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

```
## Step 10: Verify the cluster and test (master node)
```shell
kubectl get pods -n kube-system
kubectl get nodes

```
## Step 11: Deploy test application on cluster (master node)
```shell
kubectl run nginx --image=nginx

```
