📘 Explanation (Memorize‑Ready)  
Pod: Runs nginx, explicitly exposes containerPort: 80.  
Service: Provides stable DNS (nginx-svc) and routes traffic to Pods with app: nginx.  
Client Pod: Labeled access: granted so it matches the NetworkPolicy rule.  
NetworkPolicy: Allows ingress only from Pods with access: granted label, on TCP port 80.  

🚨Trap Radar  
Forgetting containerPort → Service has no endpoints → “Connection refused.”  
Pod label mismatch → Service shows <none> endpoints.  
Using Pod name instead of Service name → “bad address.”  
Wrong namespace → Policy ignored.  
Missing policyTypes → incomplete policy.  

> [!IMPORTANT]
> ```bash
> kubectl label pod nginx-pod -n web-ns app=nginx --overwrite
> ```  

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
    image: nginx
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
  - name: busybox
    image: busybox
    command: ["sleep","3600"]

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
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
# Test from disallowed pod (should fail)
kubectl run test-pod -n web-ns --image=busybox --restart=Never --command -- wget -qO- http://nginx-svc:80
```

