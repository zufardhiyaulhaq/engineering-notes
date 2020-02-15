# Install Kubernetes with Minikube
To install kubernetes using minikube, you must install the executable minikube tools first, I am using macOS
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
  && chmod +x minikube
sudo mv minikube /usr/local/bin
```
for linux user
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```
you also need Virtualbox.
### Start Minikube
- Using spesific kubernetes version
```
minikube start --kubernetes-version=1.16.6
```
- Using Virtualbox driver
```
minikube start --vm-driver='virtualbox'
```
- Using Spesific spec
```
minikube start --kubernetes-version=1.16.6 --disk-size='30000mb' --cpus=2 --memory='4096mb' --vm-driver='virtualbox'
```
### Other Useful command
- list all addons minikube
```
minikube addons list
```
- enable addons
```
minikube addons enable <name>
```
- ssh into kubernetes vm
```
minikube ssh
```
- Open kubernetes dashboard
```
minikube addons enable dashboard
minikube dashboard
```
- stop minikube
```
minikube stop
```
- delete minikube
```
minikube delete
```
