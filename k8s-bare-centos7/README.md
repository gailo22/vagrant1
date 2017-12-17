## Kubernetes on bare metal Centos 7 with 3 nodes cluster

#### Install docker

```
sudo su -
yum install -y docker
systemctl enable docker && systemctl start docker
```

#### Install kubeadm, kubelet and kubectl

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

```

#### Disable firewall and selinux
```
vi /etc/sysconfig/selinux
systemctl disable firewalld
yum remove chrony -y
yum install ntp -y
systemctl enable ntpd.service
systemctl start ntpd.service
swapoff -a
vi /etc/fstab
vi /etc/sysctl.conf
# add
net.bridge.bridge-nf-call-iptables = 1
sysctl -p
```

#### Initialize the cluster

```
kubeadm reset
systemctl daemon-reload
systemctl restart kubelet

[root@kube-master ~]# kubeadm init --skip-preflight-checks --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.100.10
Flag --skip-preflight-checks has been deprecated, it is now equivalent to --ignore-preflight-errors=all
[init] Using Kubernetes version: v1.9.0
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
  [WARNING FileExisting-crictl]: crictl not found in system path
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [kube-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.100.10]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 34.003063 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node kube-master as master by adding a label and a taint
[markmaster] Master kube-master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 44b91f.fa0036e25258c0d4
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 44b91f.fa0036e25258c0d4 192.168.100.10:6443 --discovery-token-ca-cert-hash sha256:0ecb4332f59ae042c959fcb477576c974cd637038188819cabe0cf875ad90afa

[root@kube-master ~]#
[root@kube-master ~]#  👍 5

[vagrant@kube-master ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                  READY     STATUS    RESTARTS   AGE
kube-system   etcd-kube-master                      1/1       Running   0          2m
kube-system   kube-apiserver-kube-master            1/1       Running   0          2m
kube-system   kube-controller-manager-kube-master   1/1       Running   0          2m
kube-system   kube-dns-6f4fd4bdf-lftp7              0/3       Pending   0          3m
kube-system   kube-proxy-hx97h                      1/1       Running   0          3m
kube-system   kube-scheduler-kube-master            1/1       Running   0          2m

```

#### Install a pod network - Fannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

[vagrant@kube-master ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                  READY     STATUS    RESTARTS   AGE
kube-system   etcd-kube-master                      1/1       Running   0          3m
kube-system   kube-apiserver-kube-master            1/1       Running   0          2m
kube-system   kube-controller-manager-kube-master   1/1       Running   0          2m
kube-system   kube-dns-6f4fd4bdf-2cpn5              3/3       Running   0          3m
kube-system   kube-flannel-ds-jhhp7                 1/1       Running   0          1m
kube-system   kube-proxy-bprkb                      1/1       Running   0          3m
kube-system   kube-scheduler-kube-master            1/1       Running   0          2m
```

#### Join cluster from nodes
```
[root@kube-node-1 ~]# kubeadm join --token 44b91f.fa0036e25258c0d4 192.168.100.10:6443 --discovery-token-ca-cert-hash sha256:0ecb4332f59ae042c959fcb477576c974cd637038188819cabe0cf875ad90afa

[preflight] Running pre-flight checks.
  [WARNING FileExisting-crictl]: crictl not found in system path
[discovery] Trying to connect to API Server "192.168.100.10:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://192.168.100.10:6443"
[discovery] Requesting info from "https://192.168.100.10:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.100.10:6443"
[discovery] Successfully established connection with API Server "192.168.100.10:6443"

This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.


vagrant@kube-master ~]$ kubectl get nodes
NAME          STATUS    ROLES     AGE       VERSION
kube-master   Ready     master    4m        v1.9.0
kube-node-1   Ready     <none>    2m        v1.9.0
kube-node-2   Ready     <none>    1m        v1.9.0

```

### Install microservices demo application
```
git clone https://github.com/gailo22/microservices-demo.git

[vagrant@kube-master kubernetes]$ ls
README.md    complete-demo.yaml  manifests           manifests-logging     manifests-policy  terraform
autoscaling  helm-chart          manifests-alerting  manifests-monitoring  manifests-zipkin
[vagrant@kube-master kubernetes]$ pwd
/home/vagrant/microservices-demo/deploy/kubernetes
[vagrant@kube-master kubernetes]$ kubectl create namespace sock-shop
namespace "sock-shop" created
[vagrant@kube-master kubernetes]$ kubectl apply -f complete-demo.yaml
deployment "carts-db" created
service "carts-db" created
deployment "carts" created
service "carts" created
deployment "catalogue-db" created
service "catalogue-db" created
deployment "catalogue" created
service "catalogue" created
deployment "front-end" created
service "front-end" created
deployment "orders-db" created
service "orders-db" created
deployment "orders" created
service "orders" created
deployment "payment" created
service "payment" created
deployment "queue-master" created
service "queue-master" created
deployment "rabbitmq" created
service "rabbitmq" created
deployment "shipping" created
service "shipping" created
deployment "user-db" created
service "user-db" created
deployment "user" created
service "user" created


[vagrant@kube-master kubernetes]$ kubectl get pods -n sock-shop
NAME                            READY     STATUS    RESTARTS   AGE
carts-5fffc8d5c5-xttkj          1/1       Running   0          5m
carts-db-7fcddfbc79-w4rrq       1/1       Running   0          5m
catalogue-676d4b9f7c-r65lx      1/1       Running   0          5m
catalogue-db-5c67cdc8cd-ts5p7   1/1       Running   0          5m
front-end-977bfd86-w8r7c        1/1       Running   0          5m
orders-65bc8c67b8-s6rw7         1/1       Running   0          5m
orders-db-775655b675-pl88q      1/1       Running   0          5m
payment-75f75b467f-7mrtw        1/1       Running   0          5m
queue-master-5c86964795-wm66b   1/1       Running   0          5m
rabbitmq-96d887875-2j2fq        1/1       Running   0          5m
shipping-7579b4bd66-lrtsf       1/1       Running   0          5m
user-5c95545647-tzhq8           1/1       Running   0          5m
user-db-5f9d89bbbb-mx7sw        1/1       Running   0          5m

[vagrant@kube-master kubernetes]$ kubectl get services -n sock-shop
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
carts          ClusterIP   10.100.254.99    <none>        80/TCP         13m
carts-db       ClusterIP   10.99.249.208    <none>        27017/TCP      13m
catalogue      ClusterIP   10.101.210.145   <none>        80/TCP         13m
catalogue-db   ClusterIP   10.102.26.50     <none>        3306/TCP       13m
front-end      NodePort    10.102.164.10    <none>        80:30001/TCP   13m
orders         ClusterIP   10.99.119.253    <none>        80/TCP         13m
orders-db      ClusterIP   10.106.15.21     <none>        27017/TCP      13m
payment        ClusterIP   10.97.186.194    <none>        80/TCP         13m
queue-master   ClusterIP   10.109.176.90    <none>        80/TCP         13m
rabbitmq       ClusterIP   10.108.34.18     <none>        5672/TCP       13m
shipping       ClusterIP   10.96.115.151    <none>        80/TCP         13m
user           ClusterIP   10.103.90.118    <none>        80/TCP         13m
user-db        ClusterIP   10.104.56.166    <none>        27017/TCP      13m
```




#### Access the microservices web app
```
[vagrant@kube-master kubernetes]$ kubectl get pods -n sock-shop -o wide
NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
carts-5fffc8d5c5-xttkj          1/1       Running   0          1h        10.244.1.2   kube-node-1
carts-db-7fcddfbc79-w4rrq       1/1       Running   0          1h        10.244.2.2   kube-node-2
catalogue-676d4b9f7c-r65lx      1/1       Running   0          1h        10.244.1.4   kube-node-1
catalogue-db-5c67cdc8cd-ts5p7   1/1       Running   0          1h        10.244.2.3   kube-node-2
front-end-977bfd86-w8r7c        1/1       Running   0          1h        10.244.2.4   kube-node-2
orders-65bc8c67b8-s6rw7         1/1       Running   0          1h        10.244.2.7   kube-node-2
orders-db-775655b675-pl88q      1/1       Running   0          1h        10.244.1.3   kube-node-1
payment-75f75b467f-7mrtw        1/1       Running   0          1h        10.244.1.5   kube-node-1
queue-master-5c86964795-wm66b   1/1       Running   0          1h        10.244.2.6   kube-node-2
rabbitmq-96d887875-2j2fq        1/1       Running   0          1h        10.244.1.6   kube-node-1
shipping-7579b4bd66-lrtsf       1/1       Running   0          1h        10.244.2.5   kube-node-2
user-5c95545647-tzhq8           1/1       Running   0          1h        10.244.2.8   kube-node-2
user-db-5f9d89bbbb-mx7sw        1/1       Running   0          1h        10.244.1.7   kube-node-1
```
open `http://192.168.100.12:30001/index.html`