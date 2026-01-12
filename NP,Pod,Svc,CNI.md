Flannel ignores NetworkPolicy → traffic flows when it should be blocked.
Calico enforces NetworkPolicy → traffic blocked as expected.

> [!NOTE]
> How to install/uninstall Calico
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
kubectl get pods -n kube-system
kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```
> [!NOTE]
> How to install/unistall Flannel
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
kubectl get pods -n kube-system
kubectl delete -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: web-ns
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: web-ns
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: web-ns
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  namespace: web-ns
  labels:
    access: granted
spec:
  containers:
  - name: curl
    image: curlimages/curl:latest
    command: ["sleep","3600"]
---
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  namespace: web-ns
  labels:
    access: granted
spec:
  containers:
  - name: curl
    image: curlimages/curl:latest
    command: ["sleep","3600"]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-nginx
  namespace: web-ns
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: granted
    ports:
    - protocol: TCP
      port: 80
```

```bash
kubectl exec -n web-ns client-pod -- curl -s http://nginx-svc:80
```

