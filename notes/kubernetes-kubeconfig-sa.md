# Create Kubeconfig from ServiceAccount

- Create `production` namespace
```
kubectl create namespace wordpress
```
- Apply below command to kubernetes, create production-sa service account and grant full access to production namespace.
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: production-sa
  namespace: production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: production-rw-role
  namespace: production
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: production-rw-rolebinding
  namespace: production
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: production-rw-role
subjects:
- namespace: production
  kind: ServiceAccount
  name: production-sa
```

- Get all need information from service account
```
NAMESPACE=production
SERVICE_ACCOUNT=production-sa
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
SECRET_NAME=$(kubectl get serviceaccount -n $NAMESPACE default -o jsonpath='{.secrets[0].name}')
TOKEN=$(kubectl get secret -n $NAMESPACE $SECRET_NAME -o jsonpath='{.data.token}' | base64 --decode)
CA=$(kubectl get secret -n $NAMESPACE $SECRET_NAME -o jsonpath='{.data.ca\.crt}')
```

- Generate kubeconfig
```
echo "
apiVersion: v1
kind: Config
clusters:
- name: ${NAMESPACE}-cluster
  cluster:
    certificate-authority-data: ${CA}
    server: ${APISERVER}
contexts:
- name: ${NAMESPACE}-context
  context:
    cluster: ${NAMESPACE}-cluster
    namespace: ${NAMESPACE}
    user: ${NAMESPACE}-user
current-context: ${NAMESPACE}-context
users:
- name: ${NAMESPACE}-user
  user:
    token: ${TOKEN}
" > $NAMESPACE.kubeconfig
```
