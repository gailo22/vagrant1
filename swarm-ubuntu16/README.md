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
