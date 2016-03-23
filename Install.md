## Install OpenShift

#####Ansible#####
Ansible could be used to install OpenShift from one host remotely.
> Note: Bellow steps ONLY needed where installing ansible.

######SSH access######
```
ssh-keygen

for host in master2.ceyes.os \
            node3.ceyes.os \
            node4.ceyes.os; \
  do ssh-copy-id -i ~/.ssh/id_rsa.pub $host; \
done
```

######Install ansible######
```
sudo yum -y install \
  https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
sudo sed -i -e "s/^enabled=1/enabled=0/" /etc/yum.repos.d/epel.repo
sudo yum -y --enablerepo=epel install ansible
```

######Install OpenShift######
```
git clone https://github.com/cPAAS/openshift-ansible

cd openshift-ansible
ansible-playbook -i inventory/cpass/hosts.byo \
  playbooks/byo/config.yml --ask-sudo-pass
```

```
oc edit node master2.ceyes.os
  spec:
    externalID: master2.ceyes.os
    #unschedulable: true
```

######Uninstall OpenShift######
```
ansible-playbook -i inventory/cpass/hosts.byo \
  playbooks/adhoc/uninstall.yml --ask-sudo-pass
```
