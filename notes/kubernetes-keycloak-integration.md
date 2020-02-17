# Integrating Kubernetes with Keycloak
### requirement
- Running Kubernetes cluster
- Running [Keycloak server (SSL)](./keycloak-installation.md)

## Integrating

#### Keycloak server
- in keycloak server, create new realm name `IAM`.
- in keycloak server, add groups `kubernetes-admin` and `kubernetes-viewer`.
- Add users `admin-user` assigned to group `kubernetes-admin` and `read-only-user` assigned to group `kubernetes-viewer`
- add password for each user
- Add client called `kubernetes`

| option | value |
| - | - |
| Client ID | kubernetes |

- edit client kubernetes

| option | value |
| - | - |
| Access type | confidential |
| Valid Redirect URIs | * |

- For `kubernetes` client create protocol mapper called `user_groups`

| option | value |
| - | - |
| name | user_groups |
| mapper_type | Group Membership |
| Token Claim Name | user_groups |

- testing

| option | value |
| - | - |
| username | username |
| password | password |
| client_secret | secret |

***note***, for client_secret, get the secret in client > credential.
```
curl \
    -d "grant_type=password" \
    -d "client_id=kubernetes" \
    -d "client_secret=<CLIENT_SECRET>" \
    -d "username=<USER_NAME>" \
    -d "password=<USER_PASSWORD>" \
    https://<KEYCLOAK_URL>:<KEYCLOAK_PORT/auth/realms/<REALM>/protocol/openid-connect/token -k
```
for example:
```
curl \
    -d "grant_type=password" \
    -d "client_id=kubernetes" \
    -d "client_secret=eef3e405-76d3-4e7d-bc2b-8597cd447ca8" \
    -d "username=admin-user" \
    -d "password=admin-password" \
    https://10.102.102.50:8443/auth/realms/IAM/protocol/openid-connect/token -k
```

to inspect jwt token
```
curl \
    --user "kubernetes:<CLIENT_SECRET>" \
    -d "token=<ACCESS_TOKEN>" \
    https://<KEYCLOAK_URL>:<KEYCLOAK_PORT>/auth/realms/<REALM>/protocol/openid-connect/token/introspect -k
```
#### Kubernetes Cluster
- copy keycloak CA into kubernetes
```
cd /opt/kc-certificate
scp keycloak.crt ubuntu@K8S-MASTER:~/
```
- move KC-CA in kubernetes node to spesific folder (I am using kubernetes hardway, maybe different)
```
sudo cp keycloak.crt /var/lib/kubernetes/
```
- add hostname dns mapping for keycloak
```
vi /etc/hosts

10.102.102.50 keycloak.zufar.io
```
- edit kubernetes API
to enable using OIDC with keycloak, you must add options in all apiserver, I am using kubernetes the hardway, simply add this into apiserver service in all master.
```
vi /etc/systemd/system/kube-apiserver.service

--oidc-issuer-url=https://<KEYCLOAK_URL>:<KEYCLOAK_PORT>/auth/realms/<REALM> \
--oidc-client-id=kubernetes \
--oidc-groups-claim=user_groups \
--oidc-username-claim=preferred_username \
--oidc-groups-prefix="oidc:" \
--oidc-username-prefix="oidc:" \
--oidc-ca-file=/var/lib/kubernetes/keycloak.crt \
```
for example:
```
--oidc-issuer-url=https://keycloak.zufar.io:8443/auth/realms/IAM \
--oidc-client-id=kubernetes \
--oidc-username-claim=preferred_username \
--oidc-groups-claim=user_groups \
--oidc-groups-prefix="oidc:" \
--oidc-username-prefix="oidc:" \
--oidc-ca-file=/var/lib/kubernetes/keycloak.crt \
```
- restart service
```
systemctl daemon-reload
systemctl restart kube-apiserver
systemctl status kube-apiserver
```
- Configure RBAC for each group
```
cat <<EOF | kubectl apply -f -
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: oidc-kubernetes-admin
subjects:
- kind: Group
  name: oidc:/kubernetes-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: oidc-kubernetes-view
subjects:
- kind: Group
  name: oidc:/kubernetes-viewer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: system:public-info-viewer
  apiGroup: rbac.authorization.k8s.io  
EOF
```
- if you using other node, make sure you have kubeconfig already. based on [k8s-keycloak-oidc-helper](./script/keycloak-kubectl.sh), edit the script environment and run
```
vi keycloak-kubectl.sh
```
```
KEYCLOAK_USERNAME=admin-user
KEYCLOAK_PASSWORD=admin-password
KEYCLOAK_URL=https://keycloak.zufar.io:8443
KEYCLOAK_REALM=IAM
KEYCLOAK_CLIENT_ID=kubernetes
KEYCLOAK_CLIENT_SECRET=eef3e405-76d3-4e7d-bc2b-8597cd447ca8
```
copy & execute the command from the bash
```
apt install jq
bash keycloak-kubectl.sh
```
set the cluster to use the spesific user
```
kubectl config set-context default --cluster=cluster-name --user=admin-user
```
- Testing
```
root@k8s-master1:~# kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
10.102.102.20   Ready    master   28h   v1.15.5
10.102.102.21   Ready    master   28h   v1.15.5
10.102.102.22   Ready    master   28h   v1.15.5
10.102.102.30   Ready    worker   28h   v1.15.5
10.102.102.31   Ready    worker   28h   v1.15.5
10.102.102.32   Ready    worker   28h   v1.15.5
10.102.102.33   Ready    worker   28h   v1.15.5
```