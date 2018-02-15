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
