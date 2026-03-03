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

```yaml
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
```

Security context in pod

```yaml
--eg1
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext: --container level is more pioritized
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
--eg2
apiVersion: v1
kind: Pod
metadata:
  name: cap-test
spec:
  containers:
    - name: tester
      image: ubuntu
      command: ["sh", "-c", "sleep infinity"]
      securityContext:
        capabilities:
          add:
            - NET_ADMIN
            - SYS_TIME
        privileged: false
```

network policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
    - Egress
      #ingress:
      #- from:
      #- ipBlock:
      #   cidr: 172.17.0.0/16
      #    except:
      #    - 172.17.1.0/24
      # - namespaceSelector:
      #    matchLabels:
      #      project: myproject
      #- podSelector:
      #    matchLabels:
      #      role: frontend
      #ports:
      #- protocol: TCP
      #  port: 6379
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - protocol: TCP
          port: 8080
    - to:
        - podSelector:
            matchLabels:
              name: mysql
      ports:
        - protocol: TCP
          port: 3306
```

---

Installing k8s using kubeadm

run on controlplane and node01

```yaml
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward=1
EOF

#Apply sysctl params without reboot

sudo sysctl --system

#verify that net.ipv4.ip_forward is set to 1 with:
sysctl net.ipv4.ip_forward

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

# To see the new version labels
sudo apt-cache madison kubeadm

sudo apt-get install -y kubelet=1.34.0-1.1 kubeadm=1.34.0-1.1 kubectl=1.34.0-1.1

sudo apt-mark hold kubelet kubeadm kubectl
```

```bash
IP_ADDR=$(ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address $IP_ADDR --pod-network-cidr=172.17.0.0/16 --service-cidr=172.20.0.0/16

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

create token to join node01
kubeadm token create --print-join-command

this command will be appeared and then copy and paste in node01
kubeadm join 10.244.220.196:6443 --token eg7d2x.sfh7zsb77an7mgaa --discovery-token-ca-cert-hash sha256:25af9cafddc4b97c8d29ec5669957e4e65b10bc91eff63b327d2a83e089d64d5


in controlplane download flannel
curl -LO https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml

add this in downloaded yaml
net-conf.json: |
    {
      "Network": "172.17.0.0/16", # Update this to match the custom PodCIDR
      "Backend": {
        "Type": "vxlan"
      }

also add this in yaml
  args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=eth0 #<= add this line
```

Installing HELM

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

adding bitnami repo to controlplane

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

    list repo
    helm repo list
    helm install amaze-surf bitnami/apache
    helm list
 helm uninstall happy-browse
 helm repo list
 helm repo remove hashicorp

 chek history
 helm history dazzling-web

 upgrade nignx
 helm upgrade dazzling-web bitnami/nginx --version 18.3.6
```

###Kustomization

<details>
<summary>Kustomization</summary>

```bash
controlplane ~/code/k8s ➜  ls
db  kustomization.yaml  message-broker  nginx
```

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/db-config.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - message-broker/rabbitmq-config.yaml
  - message-broker/rabbitmq-depl.yaml
  - message-broker/rabbitmq-service.yaml
  - nginx/nginx-depl.yaml
  - nginx/nginx-service.yaml
```

```bash
controlplane ~/code/k8s ➜  kubectl apply -k .
```

You can divide like this
└── k8s
├── db
│ └── kustomization.yaml
├── kustomization.yaml
├── message-broker
│ └── kustomization.yaml
└── nginx
└── kustomization.yaml
In /k8s/kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - message-broker/
  - nginx/
```

In /k8s/db/kustomization.yaml

```yaml
resources:
  - db-config.yaml
  - db-depl.yaml
  - db-service.yaml
```

In /k8s/message-broker/kustomization.yaml

```yaml
resources:
  - rabbitmq-config.yaml
  - rabbitmq-depl.yaml
  - rabbitmq-service.yaml
```

In /k8s/nginx/kustomization.yaml

```yaml
resources:
  - nginx-depl.yaml
  - nginx-service.yaml
```

use this to create kustomization.yaml

```bash
controlplane code/k8s/message-broker ➜  kustomize create
```

</details>

> [!NOTE]
> This is a note callout.

> [!TIP]
> This is a tip callout.

> [!WARNING]
> This is a warning callout.

> [!IMPORTANT]
> This is an important callout.

> [!CAUTION]
> This is a caution callout.
