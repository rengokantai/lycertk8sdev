# lycertk8sdev
use flannel. master
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```
copy config file and set previleges
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```
get flannel map.
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
kubectl get pods --all-namespaces
```
slave:
```
kubeadm join --token x master:port --discovery-token-ca-cert-hash sha256:
```
two kinds of name:
- kube-proxy
- kube-flannel

###
arch, master
- kube-apiserver, use etcd a kv pair to store data
- kube-scheduler. which node to response the request
- cloud-controller-manager. persistent storage, route, netorking
- kube-controller-manager

arch, slave
- kubelet
- kube proxy
- pod (container)

### 
```
cat << EOF > /etc/docker/daemon.json
{
  "exec-opts":["native.cgroupdriver=systemd"]
}
EOF
```
```
curl -s https://packages.cloud.google.com/apt/doc/pt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
```
apt update
apt install -y kubelet kubeadm kubectl
```

start
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```


Node controller
- Assign CIDR block to a newly registered node
- keeps track of the nodes
- monitors the node health
- evicts pods from unhealthy nodes
- can taint nodes based on current conditions in more recent versions











### Securing Images
note
- Regularly apply security updates to your environment
- Dont run tools like apt-update in containers
- Use rolling updates
- Ensure only authorized images are used in your environment
- Use private registries to store approved images
- CI pipeline should be ensure that only vetted code is used for building images

