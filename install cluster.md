************* Install Kubernetes on Master Node *************

Upgrade apt packages
```shell
sudo apt-get update
```
Create a configuration file for containerd:

```shell
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
Load modules:
```shell
sudo modprobe overlay
sudo modprobe br_netfilter
```

Set system configurations for Kubernetes networking:
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

Install containerd:
```shell
sudo apt-get update && sudo apt-get install -y containerd
```

Create a default configuration file for containerd:
```shell
sudo mkdir -p /etc/containerd
```

Generate default containerd configuration and save to the newly created default file:
```shell
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Restart containerd to ensure new configuration file usage:
```shell
sudo systemctl restart containerd
```
Verify that containerd is running.
```shell
sudo systemctl status containerd
```

Disable swap:
```shell
sudo swapoff -a
```

Disable swap on startup in /etc/fstab:
```shell
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Install dependency packages:
```shell
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
```

Install kub:
```shell
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update

sudo apt install -y kubeadm=1.28.9-2.1 kubelet=1.28.9-2.1 kubectl=1.28.9-2.1

```


Turn off automatic updates:

```shell
sudo apt-mark hold kubelet kubeadm kubectl
```

Log into both Worker Nodes to perform previous steps 1 to 18.
Initialize the Cluster-

Initialize the Kubernetes cluster on the control plane node using kubeadm (Note: This is only performed on the Control Plane Node):

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