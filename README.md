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
