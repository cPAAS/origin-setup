# OpenShift Origin setup

## Prerequisites

#####Base OS#####

```
Centos 7.1 with "Minimal" installation option
```

#####Package update#####

```
sudo yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion
sudo yum -y update
```

#####DNS#####

setup the DNS server(bind) or use your router to add DNS entry for each of your nodes

```
master1.ceyes.os (192.168.0.141)
node1.ceyes.os (192.168.0.144)
```

create a wildcard DNS entry for your apps and points to the public IP address of the host where the router will be deployed
```
*.ceyes.os (192.168.0.141)
```

#####SSH access#####

on master
```
ssh-keygen
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
ssh-copy-id -i ~/.ssh/id_rsa.pub node1.ceyes.os

sudo vi /etc/sysconfig/iptables
  -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
  -A INPUT -p tcp -m state --state NEW -m tcp --dport 8443 -j ACCEPT
sudo service iptables restart
```

#####Docker#####

Installing Docker
```
sudo yum -y install docker
sudo vi /etc/sysconfig/docker
OPTIONS='--selinux-enabled --insecure-registry 172.30.0.0/16'
```

Configuring Docker Storage
```
# create docker-vg (LVM) partition during Centos installation

sudo vi /etc/sysconfig/docker-storage-setup
  VG=docker-vg
sudo docker-storage-setup

sudo systemctl enable docker.service
sudo rm -rf /var/lib/docker/*
sudo systemctl start docker.service
```

Enabling Docker service when boot
```
sudo chkconfig docker on
```

#####Misc Q&A#####

Q1: How to enable network when boot
```
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
  ONBOOT="yes"
```

Q2: How to start docker service when boot
```
sudo chkconfig docker on
```

Q3: How to running `docker` command without `sudo`
```
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker.service
```

Q4: How to debug LVolume issues
```
sudo lvdisplay
sudo lvremove -f docker-vg
sudo umount /dev/docker-vg/data
sudo lvremove -f docker-vg
```

Q5: yum install reporting “Another app is currently holding the yum lock; waiting for it to exit...”
```
# should be due to yumBackend.py running in bg
sudo yum remove PackageKit
```
