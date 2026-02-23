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

you gonna need this when you install nginx-gateway
30080 → often used as a NodePort for HTTP (port 80)
30081 → often used as a NodePort for HTTPS (port 443)
k edit svc nginx-gateway -n nginx-gateway

---

Maintenance

Step 3: Update the etcd configuration

Open the etcd configuration file for editing:

vi /etc/kubernetes/manifests/etcd.yaml
Modify the volumes section as follows:

From:

volumes:

- hostPath:
  path: /etc/kubernetes/pki/etcd
  type: DirectoryOrCreate
  name: etcd-certs
- hostPath:
  path: /var/lib/etcd # OLD directory
  type: DirectoryOrCreate
  name: etcd-data
  To:

volumes:

- hostPath:
  path: /etc/kubernetes/pki/etcd
  type: DirectoryOrCreate
  name: etcd-certs
- hostPath:
  path: /var/lib/etcd-from-backup # NEW restored directory
  type: DirectoryOrCreate
  name: etcd-data

Security
read certificate
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

to wrap base64
cat akshay.csr | base64 --w 0

config file with dir
kubectl config --kubeconfig=/root/my-kube-config use-context research

**multi rule in role**
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
creationTimestamp: "2026-02-23T15:32:45Z"
name: developer
namespace: blue
resourceVersion: "3211"
uid: 2d24db8f-4778-4979-bfd4-9ffbf84ec787
rules:

- apiGroups:
  - ""
    resourceNames:
  - dark-blue-app
    resources:
  - pods
    verbs:
  - get
  - watch
  - create
  - delete
- apiGroups:
  - apps
    resources:
  - deployments
    verbs:
  - get
  - watch
  - create
  - delete~
