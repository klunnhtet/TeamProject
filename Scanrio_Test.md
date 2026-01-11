---
## 🧪 Scenario > In the `dev` namespace: 
> - Create a Secret named `db-secret` 
> - Deploy a Pod named `web-pod` using a custom ServiceAccount
> - Grant the Pod permission to read the Secret using Role and RoleBinding
---
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---

apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: dev
type: Opaque
stringData:
  password: s3cr3t
---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: web-sa
  namespace: dev
---

apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  namespace: dev
spec:
  serviceAccountName: web-sa
  containers:
  - name: nginx
    image: nginx
    command: ["sleep", "3600"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: web-sa
  namespace: dev
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
---

kubectl -n dev exec -it web-pod -- sh

TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -sSk -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/dev/secrets/db-secret | jq
```
