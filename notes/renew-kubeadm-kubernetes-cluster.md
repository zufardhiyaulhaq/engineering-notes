# Renew Kubeadm Kubernetes cluster certificate
- to check certificate expire in master
```
kubeadm alpha certs check-expiration
```
- to renew
```
kubeadm alpha certs renew
```

### logs
```
root@k8s-community-master:~# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Feb 11, 2021 11:11 UTC   364d                                    no
apiserver                  Feb 11, 2021 11:11 UTC   364d            ca                      no
apiserver-etcd-client      Feb 11, 2021 11:11 UTC   364d            etcd-ca                 no
apiserver-kubelet-client   Feb 11, 2021 11:11 UTC   364d            ca                      no
controller-manager.conf    Feb 11, 2021 11:11 UTC   364d                                    no
etcd-healthcheck-client    Feb 11, 2021 11:11 UTC   364d            etcd-ca                 no
etcd-peer                  Feb 11, 2021 11:11 UTC   364d            etcd-ca                 no
etcd-server                Feb 11, 2021 11:12 UTC   364d            etcd-ca                 no
front-proxy-client         Feb 11, 2021 11:12 UTC   364d            front-proxy-ca          no
scheduler.conf             Feb 11, 2021 11:12 UTC   364d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Feb 07, 2030 08:59 UTC   9y              no
etcd-ca                 Feb 07, 2030 08:59 UTC   9y              no
front-proxy-ca          Feb 07, 2030 08:59 UTC   9y              no
root@k8s-community-master:~# kubeadm alpha certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
root@k8s-community-master:~# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Feb 11, 2021 11:42 UTC   364d                                    no
apiserver                  Feb 11, 2021 11:42 UTC   364d            ca                      no
apiserver-etcd-client      Feb 11, 2021 11:42 UTC   364d            etcd-ca                 no
apiserver-kubelet-client   Feb 11, 2021 11:42 UTC   364d            ca                      no
controller-manager.conf    Feb 11, 2021 11:42 UTC   364d                                    no
etcd-healthcheck-client    Feb 11, 2021 11:42 UTC   364d            etcd-ca                 no
etcd-peer                  Feb 11, 2021 11:42 UTC   364d            etcd-ca                 no
etcd-server                Feb 11, 2021 11:42 UTC   364d            etcd-ca                 no
front-proxy-client         Feb 11, 2021 11:42 UTC   364d            front-proxy-ca          no
scheduler.conf             Feb 11, 2021 11:42 UTC   364d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Feb 07, 2030 08:59 UTC   9y              no
etcd-ca                 Feb 07, 2030 08:59 UTC   9y              no
front-proxy-ca          Feb 07, 2030 08:59 UTC   9y              no
root@k8s-community-master:~#
```
***Notes***: this command only renew certificate in master node