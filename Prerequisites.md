# OpenShift Origin setup

## Prerequisites

#####Base OS#####

```
Centos 7.1 with "Minimal" installation option
```

#####Package update#####

```
# Download http://mirrors.163.com/.help/CentOS7-Base-163.repo
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
mv CentOS7-Base-163.repo /etc/yum.repos.d/CentOS-Base.repo
sudo yum clean all
sudo yum makecache
```

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

#####Registry#####

Running registry

```
docker pull registry:2
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/regvol:/var/lib/registry:z registry:2
```

> **About the option `z`**
> 
> without `z`, when push images to registry it will report bellow error
> 
> ```
> filesystem: mkdir /var/lib/registry/docker: permission denied
> ```
> 
> ```
> drwxrwxrwx.  2 1000 1000    6 Mar 17 06:41 /var/lib/registry
> ```
> 
> It's the [seLinux issue](http://www.projectatomic.io/blog/2015/06/using-volumes-with-docker-can-cause-problems-with-selinux/), that the problem here was the user created a volume and labeled it from one container with the "Z" for one container, then attempted to share it in another container. Which SELinux denied since the Multi-Category Security (MCS) labels differed.
> 
> Solution: if you volume mount a image with `-v /SOURCE:/DESTINATION:z` docker will automatically relabel the content for you to s0.

#####Misc Q&A#####

**Q1: How to enable network when boot**
```
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
  ONBOOT="yes"
```

**Q2: How to start docker service when boot**
```
sudo chkconfig docker on
```

**Q3: How to running `docker` command without `sudo`**
```
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker.service
```

**Q4: How to debug LVolume issues**
```
sudo lvdisplay
sudo lvremove -f docker-vg
sudo umount /dev/docker-vg/data
sudo lvremove -f docker-vg
```

**Q5: yum install reporting “Another app is currently holding the yum lock; waiting for it to exit...”**
```
# should be due to yumBackend.py running in bg
sudo yum remove PackageKit
```

**Q6: How to play with LVM**

######create partition######
```
# fdisk -l
# fdisk /dev/sda

Command (m for help): n
Partition type:
   p   primary (3 primary, 0 extended, 1 free)
   e   extended
Select (default e): p
Selected partition 4
First sector (177192960-209715199, default 177192960): 
Using default value 177192960
Last sector, +sectors or +size{K,M,G} (177192960-209715199, default 209715199): 
Using default value 209715199
Partition 4 of type Linux and of size 15.5 GiB is set

Command (m for help): t
Partition number (1-4, default 4): 
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    93306879    46140416   8e  Linux LVM
/dev/sda3        93306880   177192959    41943040   8e  Linux LVM
/dev/sda4       177192960   209715199    16261120   8e  Linux LVM

Command (m for help): w
WARNING: Re-reading the partition table failed with erro 16: Device or resource busy
```

######apply the partion info wihtout reboot######
```
# partprobe
```

######create PV and VG######
```
# pvcreate /dev/sda3
# vgcreate docker-vg /dev/sda3
```

######create LV######
```
lvcreate -L 100M -n docker docker-vg
```
