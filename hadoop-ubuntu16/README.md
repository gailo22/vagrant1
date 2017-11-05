** Hadoop via docker

<code>
ubuntu@ubuntu-xenial:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
ubuntu@ubuntu-xenial:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
ubuntu@ubuntu-xenial:~$
ubuntu@ubuntu-xenial:~$ sudo apt-get update
ubuntu@ubuntu-xenial:~$ sudo apt-get install -y docker-ce
ubuntu@ubuntu-xenial:~$ sudo systemctl status docker
ubuntu@ubuntu-xenial:~$ sudo usermod -aG docker ${USER}
ubuntu@ubuntu-xenial:~$ docker ps

</code>
