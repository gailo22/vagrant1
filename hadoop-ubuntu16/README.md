# Hadoop via docker

```
ubuntu@ubuntu-xenial:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
ubuntu@ubuntu-xenial:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
ubuntu@ubuntu-xenial:~$
ubuntu@ubuntu-xenial:~$ sudo apt-get update
ubuntu@ubuntu-xenial:~$ sudo apt-get install -y docker-ce
ubuntu@ubuntu-xenial:~$ sudo systemctl status docker
ubuntu@ubuntu-xenial:~$ sudo usermod -aG docker ${USER}
ubuntu@ubuntu-xenial:~$ docker ps

#Run hadoop container
ubuntu@ubuntu-xenial:~$ docker pull sequenceiq/hadoop-docker:2.7.1
ubuntu@ubuntu-xenial:~$ docker run -it sequenceiq/hadoop-docker:2.7.1 /etc/bootstrap.sh -bash

#Inside the container
bash-4.1# cd $HADOOP_PREFIX
bash-4.1# bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep input output 'dfs[a-z.]+'
ubuntu@ubuntu-xenial:~$ bin/hdfs dfs -cat output/*
```
