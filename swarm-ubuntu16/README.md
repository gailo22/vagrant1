# Swarm Cluster setup

* This is to create 3 ubuntu vms to running docker swarm cluster


## Prerequisites
Install docker for all machine follow [this](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04)

## Setup swarm cluster
```
# Master node
ubuntu@master:~$ docker swarm init --advertise-addr 10.0.0.10:2377
Swarm initialized: current node (vprjzyns8sc84eq54frb11lok) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4jbdkeqy5q8mb7ivjwfecf7gkvmjug8fsm2y16hlacw2ypkmm2-a43uli2ft5veiyx6mjc4p4ebi 10.0.0.10:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# At 2 other nodes
ubuntu@node1:~$ docker swarm join --token SWMTKN-1-4jbdkeqy5q8mb7ivjwfecf7gkvmjug8fsm2y16hlacw2ypkmm2-a43uli2ft5veiyx6mjc4p4ebi 10.0.0.10:2377
This node joined a swarm as a worker.

ubuntu@node2:~$ docker swarm join --token SWMTKN-1-4jbdkeqy5q8mb7ivjwfecf7gkvmjug8fsm2y16hlacw2ypkmm2-a43uli2ft5veiyx6mjc4p4ebi 10.0.0.10:2377
This node joined a swarm as a worker.


ubuntu@master:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
vprjzyns8sc84eq54frb11lok *   master              Ready               Active              Leader
a8x4xngrjz6lson6qlun5fp4m     node1               Ready               Active
tgxsa3o0g2clky5txqrguwvy7     node2               Ready               Active

```

### Create solr network
```
ubuntu@master:~$ docker network create --driver overlay solr_net
mlxvam35e6yjuftxcjvrbpqos
```

### Start zookeeper
```
ubuntu@master:~$ docker service create --name zookeeper --replicas 1 --network solr_net jplock/zookeeper

ubuntu@master:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                     PORTS
nyl3fy8khjqm        zookeeper           replicated          1/1                 jplock/zookeeper:latest
```

### Start solr
```
ubuntu@master:~$ docker service create --name solr --replicas 2 --network solr_net -p 8983:8983 \
   solr \
   bash -c '/opt/solr/bin/solr start -f -z zookeeper:2181'

ubuntu@master:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                     PORTS
bqzp3ezvadfy        solr                replicated          2/2                 solr:latest               *:8983->8983/tcp
nyl3fy8khjqm        zookeeper           replicated          1/1                 jplock/zookeeper:latest
```

Check solr admin by open `http://10.0.0.10:8983/solr/#/~cloud` or `http://10.0.0.11:8983/solr/#/~cloud` or `http://10.0.0.12:8983/solr/#/~cloud`

### Scale the solr service
```
ubuntu@master:~$ docker service scale solr=3

ubuntu@master:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                     PORTS
bqzp3ezvadfy        solr                replicated          3/3                 solr:latest               *:8983->8983/tcp
nyl3fy8khjqm        zookeeper           replicated          1/1                 jplock/zookeeper:latest
```

You can see REPLICAS is 3/3 now.


## Setup zookeeper cluster
Previously we have only one zookeepr running. When we kill it whole solr cluster will die also.

This time we will setup zookeeper cluster so it won't be a single point of failure.
```
ubuntu@master:~$ docker stack deploy -c stack.yml zookeeper
Ignoring unsupported options: restart

Creating network zookeeper_default
Creating service zookeeper_zoo1
Creating service zookeeper_zoo2
Creating service zookeeper_zoo3

ubuntu@master:~$ docker service create --name solr --replicas 2 --network zookeeper_default -p 8983:8983 \
     solr \
     bash -c '/opt/solr/bin/solr start -f -z zookeeper_zoo1:2181'

ubuntu@master:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
l359lpddk7xf        solr                replicated          2/2                 solr:latest         *:8983->8983/tcp
oygutb6npldk        zookeeper_zoo1      replicated          1/1                 zookeeper:latest    *:2181->2181/tcp
hkovfzjrqgtq        zookeeper_zoo2      replicated          1/1                 zookeeper:latest    *:2182->2181/tcp
ckkgzr99fkwi        zookeeper_zoo3      replicated          1/1                 zookeeper:latest    *:2183->2181/tcp
```

### Just kill it
```
ubuntu@master:~$ docker kill zookeeper_zoo1.1.sfued02721v2oww1pli56g15p
zookeeper_zoo1.1.sfued02721v2oww1pli56g15p
```
It come back very fast and solr cluster still running fine. 
This is very nice.

Reference: by this [Link](https://docs.docker.com/samples/library/zookeeper/)
