## Prapare Python environment

```bash

$ vagrant  ssh  k8s-m1
[vagrant@k8s-m1 ~]$ sudo -s
[root@k8s-m1 vagrant]# cd /tmp
[root@k8s-m1 tmp]# curl -O https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh
[root@k8s-m1 tmp]# bash Anaconda3-5.3.1-Linux-x86_64.sh
[root@k8s-m1 tmp]# source /root/anaconda3/etc/profile.d/conda.sh
[root@k8s-m1 tmp]# conda env list
[root@k8s-m1 tmp]# conda remove   kybespray
[root@k8s-m1 tmp]# conda create --name kubespray  python=3.7
[root@k8s-m1 vagrant]#  conda activate kubespray
(kubespray) [root@k8s-m1 vagrant]# 
```

## Duplicate Ansible script ( kubespray )


```bash
(kubespray) [root@k8s-m1 vagrant]# yum install -y git
(kubespray) [root@k8s-m1 vagrant]# cd ~/
(kubespray) [root@k8s-m1 vagrant]#  git clone https://github.com/kubernetes-incubator/kubespray.git
(kubespray) [root@k8s-m1 vagrant]#  cd kubespray && git checkout v2.10.4
(kubespray) [root@k8s-m1 kubespray]#  pip install -r requirements.txt
(kubespray) [root@k8s-m1 kubespray]#  yum install -y epel-release && yum install -y ansible 

(kubespray) [root@k8s-m1 kubespray]# mkdir inventory/mycluster && cp -rfp inventory/sample/* inventory/mycluster

###(kubespray) [root@k8s-m1 kubespray]# rm -rf inventory/sample inventory/local/

###  Update the Ansible inventory file with inventory builder
(kubespray) [root@k8s-m1 kubespray]#  declare -a IPS=(10.100.198.201 10.100.198.202 10.100.198.203 10.100.198.211 10.100.198.212 10.100.198.213 10.100.198.221 10.100.198.222 10.100.198.223 )
(kubespray) [root@k8s-m1 kubespray]# CONFIG_FILE=inventory/mycluster/hosts.ini python contrib/inventory_builder/inventory.py ${IPS[@]}
DEBUG: Adding group all
DEBUG: Adding group kube-master
DEBUG: Adding group kube-node
DEBUG: Adding group etcd
DEBUG: Adding group k8s-cluster
DEBUG: Adding group calico-rr
DEBUG: adding host node1 to group all
DEBUG: adding host node2 to group all
DEBUG: adding host node3 to group all
DEBUG: adding host node4 to group all
DEBUG: adding host node5 to group all
DEBUG: adding host node6 to group all
DEBUG: adding host node7 to group all
DEBUG: adding host node8 to group all
DEBUG: adding host node9 to group all
DEBUG: adding host node1 to group etcd
DEBUG: adding host node2 to group etcd
DEBUG: adding host node3 to group etcd
DEBUG: adding host node1 to group kube-master
DEBUG: adding host node2 to group kube-master
DEBUG: adding host node1 to group kube-node
DEBUG: adding host node2 to group kube-node
DEBUG: adding host node3 to group kube-node
DEBUG: adding host node4 to group kube-node
DEBUG: adding host node5 to group kube-node
DEBUG: adding host node6 to group kube-node
DEBUG: adding host node7 to group kube-node
DEBUG: adding host node8 to group kube-node
DEBUG: adding host node9 to group kube-node
(kubespray) [root@k8s-m1 kubespray]#

```
- It should generate the “inventory/mycluster/hosts.ini” file with following hosts mapping, you can change it as per your need.

But the below is wrong (2.10.x) 
```
(kubespray) [root@k8s-m1 kubespray]# vi inventory/mycluster/hosts.ini

all:
  hosts:
    node1:
      ansible_host: 10.100.198.201
      ip: 10.100.198.201
      access_ip: 10.100.198.201
    node2:
      ansible_host: 10.100.198.202
      ip: 10.100.198.202
      access_ip: 10.100.198.202
    node3:
      ansible_host: 10.100.198.203
      ip: 10.100.198.203
      access_ip: 10.100.198.203
    node4:
      ansible_host: 10.100.198.211
      ip: 10.100.198.211
      access_ip: 10.100.198.211
    node5:
      ansible_host: 10.100.198.212
      ip: 10.100.198.212
      access_ip: 10.100.198.212
    node6:
      ansible_host: 10.100.198.213
      ip: 10.100.198.213
      access_ip: 10.100.198.213
    node7:
      ansible_host: 10.100.198.221
      ip: 10.100.198.221
      access_ip: 10.100.198.221
    node8:
      ansible_host: 10.100.198.222
      ip: 10.100.198.222
      access_ip: 10.100.198.222
    node9:
      ansible_host: 10.100.198.223
      ip: 10.100.198.223
      access_ip: 10.100.198.223
  children:
    kube-master:
      hosts:
        node1:
        node2:
    kube-node:
      hosts:
        node1:
        node2:
        node3:
        node4:
        node5:
        node6:
        node7:
        node8:
        node9:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
    calico-rr:
      hosts: {}
```

use 2.8.5
```bash
[all]
k8s-m1    ansible_host=10.100.198.201 ip=10.100.198.201
k8s-m2   ansible_host=10.100.198.202 ip=10.100.198.202
k8s-m3   ansible_host=10.100.198.203 ip=10.100.198.203
k8s-e1   ansible_host=10.100.198.211 ip=10.100.198.211
k8s-e2   ansible_host=10.100.198.212 ip=10.100.198.212
k8s-e3   ansible_host=10.100.198.213 ip=10.100.198.213
k8s-n1   ansible_host=10.100.198.221 ip=10.100.198.221
k8s-n2   ansible_host=10.100.198.222 ip=10.100.198.222
k8s-n3   ansible_host=10.100.198.223 ip=10.100.198.223

[kube-master]
k8s-m1
k8s-m2
k8s-m3

[kube-node]
k8s-n1
k8s-n2
k8s-n3

[etcd]
k8s-e1
k8s-e2
k8s-e3

[k8s-cluster:children]
kube-node
kube-master

[calico-rr]

[vault]
k8s-m1
k8s-m2
k8s-m3


```





```bash

$ vagrant up  k8s-m2 k8s-m3    k8s-e1 k8s-e2 k8s-e3  k8s-n1 k8s-n2 k8s-n3

```

Generate root SSH keys for  k8s-m1 and copy the keys to k8s-m2 k8s-m3    k8s-e1 k8s-e2 k8s-e3  k8s-n1 k8s-n2 k8s-n . The password for the root user on these VMs is vagrant . Enter the root user password when asked by the ssh-copy-id command and proceed with the default settings:


```bash 
$  vagrant ssh   k8s-m1
Last login: Wed Jul 17 12:10:44 2019 from 10.0.2.2
bash: /root/anaconda3/etc/profile.d/conda.sh: Permission denied
[vagrant@k8s-m1 ~]$  sudo su -
[root@k8s-m1 ~]#  ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:pAqB7R6agIo30DlUgzHClskwiL3XgpbRRah7F4GjYrQ root@k8s-m1
The key's randomart image is:
+---[RSA 2048]----+
|X.=+o=o          |
|o%+o=..          |
|+ =B o ..        |
|oEBoo oo         |
|=oBo ...S        |
|+=.+...          |
|= +...           |
| . .             |
|                 |
+----[SHA256]-----+
[root@k8s-m1 ~]#
[root@k8s-m1 ~]# ssh-copy-id k8s-m2
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-m2 (10.100.198.202)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-m2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-m2'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-m3
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-m3 (10.100.198.203)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-m3's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-m3'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-e1
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-e1 (10.100.198.211)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-e1's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-e1'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-e2
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-e2 (10.100.198.212)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-e2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-e2'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-e3
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-e3 (10.100.198.213)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-e3's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-e3'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-n1
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-n1 (10.100.198.221)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-n1's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-n1'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-n2
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-n2 (10.100.198.222)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-n2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-n2'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]# ssh-copy-id k8s-n3
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'k8s-n3 (10.100.198.223)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@k8s-n3's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'k8s-n3'"
and check to make sure that only the key(s) you wanted were added.

[root@k8s-m1 ~]#

```

9.  # Review and change parameters under ``inventory/mycluster/group_vars``
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
Some changes you may want to look for are:

-  Changingthe network as per your liking in “inventory/mycluster/group_vars/k8s-cluster.yml”

```
# Choose network plugin (cilium, calico, contiv, weave or flannel)
# Can also be set to 'cloud', which lets the cloud provider setup appropriate routing
kube_network_plugin: weave
```

- In “inventory/mycluster/group_vars/all.yml”  uncomment the following line to enable metrics to fetch the cluster resource utilization data without this HPAs will not work (for ‘kubectl top nodes’ & ‘kubectl top pods’ commands to work)

# The read-only port for the Kubelet to serve on with no authentication/authorization. Uncomment to enable.
kube_read_only_port: 10255


Now we are all set for the big, red switch:

10.   Deploy Kubespray with Ansible Playbook

```
(kubespray) [root@k8s-m1 kubespray]#   pip   install --upgrade ansible
~]# ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml  -e dns_mode='none'
~]# export ANSIBLE_INVALID_TASK_ATTRIBUTE_FAILED=False
~]# ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml  -e dns_mode='coredns'

```
