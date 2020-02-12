# Install Kubernetes with kubeadm
I have 2 nodes for kubernetes
- k8s-community-master (10.254.254.10) Ubuntu 18.04
- k8s-community-worker (10.254.254.20) Ubuntu 18.04

This cluster will using Docker as container runtime and flannel as network plugin. Lets start install them
- add hosts mapping
```
vi /etc/hosts
```
```
k8s-community-master 10.254.254.10
k8s-community-worker 10.254.254.20
```
- disable Swap
```
swapon -s
swapoff -a  
```
edit fstab (comment it using #) to permanently disable swap
```
vi /etc/fstab
```
- Install Docker
```
# Install Docker CE
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2

### Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

### Add Docker apt repository.
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

## Install Docker CE.
apt-get update && apt-get install -y \
  containerd.io=1.2.10-3 \
  docker-ce=5:19.03.4~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.4~3-0~ubuntu-$(lsb_release -cs)

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```
- Install kubeadm, kubectl, & kubelet
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
- Run bootstrap command in master node
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```
create .kube config directory
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
install flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
- Join worker node
run this command on each worker node
```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```
to get the token, run in master node
```
kubeadm token list
```
to get the ca-cert hash, run in master node
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```
*control-plane-host* is IP address or hostname of master node, *control-plane-port* default is 6443.
- Check nodes
```
kubectl get nodes
```
- Tags worker node
```
kubectl label node <node-name> node-role.kubernetes.io/worker=
```
