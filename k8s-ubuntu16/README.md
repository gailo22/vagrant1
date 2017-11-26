## Kubernetes

Start minikube
```
$ minikube start
Starting local Kubernetes v1.8.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.

$ kubectl get node
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    27d       v1.8.0

$ kubectl create -f first-app/helloworld.yml
pod "nodehelloworld.example.com" created

$ kubectl get pod
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-5bc754d4cd-qxxtk   1/1       Running             3          27d
nginx-78687579d-z4gzn             1/1       Running             2          15d
nodehelloworld.example.com        0/1       ContainerCreating   0          49s

$ kubectl describe pod nodehelloworld.example.com

$ kubectl port-forward nodehelloworld.example.com 8081:3000
Forwarding from 127.0.0.1:8081 -> 3000
Handling connection for 8081

Montree-MacBook:~ montree$ curl localhost:8081
Hello World!

ontree-MacBook:k8s-ubuntu16 montree$ kubectl expose pod nodehelloworld.example.com --type=NodePort --name nodehelloworld-service
service "nodehelloworld-service" exposed

Montree-MacBook:~ montree$ minikube service nodehelloworld-service --url
http://192.168.99.100:30298

Montree-MacBook:~ montree$ curl http://192.168.99.100:30298
Hello World!


```


Setup vagrant and install docker container
```
vagrant up
vagrant ssh

# install docker
vagrant@vagrant:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
vagrant@vagrant:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
vagrant@vagrant:~$ sudo apt-get update


vagrant@vagrant:~$ apt-cache policy docker-ce

vagrant@vagrant:~$ sudo apt-get install -y docker-ce

vagrant@vagrant:~$ sudo usermod -aG docker ${USER}
vagrant@vagrant:~$ su - ${USER}
Password:
vagrant@vagrant:~$ id -nG
vagrant adm cdrom sudo dip plugdev lxd lpadmin sambashare docker

vagrant@vagrant:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

```