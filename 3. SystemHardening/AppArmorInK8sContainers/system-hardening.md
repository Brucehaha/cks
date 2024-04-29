```shell
sudo apparmor_parser /etc/apparmor.d/deny-write
kubectl create -f cks/SystemHardening/AppArmorInK8sContainers/apparmor-disk-write-pod.yml --save-config
kubectl exec apparmor-disk-write --cat diskwrite.log
```