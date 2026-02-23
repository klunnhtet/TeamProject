Networking notes
to find default route
ip route

find kubelet service and check container runtime endpoint value
cat /var/lib/kubelet/config.yaml

kubelct config file
cat /etc/kubernetes/kubelet.conf

install Calico cni
https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

check cicd for cni like 172.17.0.0/16
cat /etc/kubernetes/manifest/kube-controller-manager.yaml

