```shell
# add --profiling=false
vim /etc/kuberntes/manifests/kube-controller-manager.yaml
# kubelet service file
vim /etc/systemd/system/kubelet.service.d/10-kubeadmin.conf
#y default, where is the kubelet configuration file located in a kubeadm cluster?
vim /var/lib/kubelet/config.yaml

```