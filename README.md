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





## Logging and Monitoring
### Monitoring Cluster and Application Components

Monitoring
- Grafana with InfluxDB
- Heapster is setup to use this storage backend by default on most Kubernetes clusters.
- InfluxDB and Grafana run in Pods
- The pod exposes itself as a Kubernetes service which is how Heapster then discover it



### Managing Logs
```
kubectl get pods --all-namespaces
```
or
```
cd /var/log/pods
```

## Cluster Maintenance
### Upgrading Kubernetes Components
master:
```
apt upgrade kubeadm
kubeadm upgrade plan
kubeadm upgrade apply v1.9.1
```
check deploy
```
kubectl get deployments
kubectl drain host1 --ignore-daemonsets  //clear pods,evict to another node
apt update
kubectl uncordon host1
kubectl drain host2 --ignore-daemonsets 
```
host2:
```
apt update && apt upgrade kubelet
```
host1(master)
```
kubectl uncordon host2 --ignore-daemonsets 
```
### Upgrading the Underlying Operating System(s)
```
kubectl drain h4 --ignore-daemonsets
kubectl delete node h4
```

token command
host1:
```
kubeadm token list
kubeadm token generate
kubeadm token create 123.456789 --ttl 3h --print-join-command
```
join master node:(copy the previous command)
```
kubeadm join --token 123.456 master:6443 --discovery-token-ca-cert-hash sha256:1234
```

## Networking
### Node Networking Configuration
#### Inbound Node Port Req
Master Node(s):
- 6443: k8s api server
- 2379-2380: etcd server client API
- 10250: kubelet API
- 10251: kube-scheduler
- 10252: kube-controller-manager
- 10255: read-only kubelet API

Worker Nodes
- 10250: Kubelet API
- 10255: Read-only Kubelet API
- 30000-32767 Nodeport services









## Storage
### Persist Volumes, Part1
kubernetes and persistent volumes
- native pod storages is ephemeral - like a pod
- what happens when a container crashes
  - kubelet restarts it(possibly on another node)
  - File system is re-created from image
  - Ephemeral files are gone
- Docker Volumes
  - Directory on disk
  - Possibly on other container
  - New Volume Drivers
- k8s volumes
  - same lifetime as its pod
  - Data preserved across container restarts
  - Pod goes away ->volume goes away
  - Directory with data
  - Accessible to containers in a pod
  - Implementation details determined by volume types
  
  
###### 08:10
note
- Created when a pod is assigned to a node
- Exists while pod runs on a paticular node
- Initially empty
- Multiple containers can read/write same volume
- Volume can be mounted per container -- same or different mount points
- Pod removed -> volume removed
- Stored on node's local medium
- Optional - set emptyDir.medium = Memory for RAM based tmpfs

sample.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: k8s.gcr.io/test-sebserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
     emptyDir: {}
```

### Persistent Volumes, Part 2
local
- Alpha, requires "PersistentLocalVolumes" and "VolumeScheduling" feature gates
- Allows local mounted storage to be mounted to a pod
- Statically created PersistentVolume
- Kubernete is aware of the volume's node constraints
- Must be on the node
- Not suitable for all applications
- 1.9+ Volume binding can be dealyed until pod scheduling








### Securing Images
note
- Regularly apply security updates to your environment
- Dont run tools like apt-update in containers
- Use rolling updates
- Ensure only authorized images are used in your environment
- Use private registries to store approved images
- CI pipeline should be ensure that only vetted code is used for building images

### Defining Security Contexts
security.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  volumes:
  - name: ke
    emptyDir: {}
  containers:
  - name: sample
    image: gcr.io/google-samples/node-hello:1.0
    volumeMounts:
    - name: ke
      mountPath: /data/demo
    securityContext:
      allowPrivateEscalation: false
```
```
kubectl create -f security.yml
kubectl get pods -o wide
kubectl exec -it security-context-pod -- sh
ps aux
cd /data/demo
exit
kubectl delete -f security.yml
```
but if we set
```
    securityContext:
      runAsUser: 1000
      allowPrivateEscalation: false
```
then the container will run as user 2000


### Troubleshooting in Kubernetes
```
kubectl get nodes -o wide
kubectl describe node hostname
kubectl logs counter
```
