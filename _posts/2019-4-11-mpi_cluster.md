---
layout: post
title: How to setup a MPI Cluster
---

A beowulf cluster is a group of general purpose systems connected over ethernet which are used for running MPI like porgrams.
Its basic software componenets are -
+ NFS - a network mounted directory which will host executable code
+ Password less SSH - so that master can execute code on slaves without asking password from user
+ OpenMPI - one of the implementations of MPI standard

## Assumptions
1. Ubuntu 16.04 will be used because it is a Long Term Support Release (5 year support) and 18.04 is not yet stable enough to be used as cluster base. OpenMPI will be used as MPI Implementation. Though, almost any combination of OS and MPI implementation can be used.
2. There will be a master (named node0) and remaining will be slaves (named as node1, node2, etc).
3. There will be an account named as "mpiuser" on all the nodes, for simplicity they all may have same password
4. There will be a directory "mpi" in "mpiuser" 's home which will be NFS mounted by all nodes. All executable files and data will have to be kept in this directory.
5. Working directory of SSH will also be mounted via NFS so as to enable launching of MPI program from any node in the cluster

## Preprocessing
1. Install same version of Ubuntu (preferably 16.04) in all the nodes (master and slaves)
2. Create a new user with name "mpiuser" on all nodes. Ensure that userID of this user is same in all machines.
3. Set the hostname of each node as per the assumption (node0, node1, etc)

## Master's (node0) Installation

### Binding hostnames with IP address
Edit /etc/hosts file. Make sure there is only one entry for 127.0.0.1 i.e. file should like-

```conf
	127.0.0.1	localhost
	192.168.17.200	node0
	192.168.17.201	node1
	192.168.17.202	node2
```

Replace these IP addresses with actual IP address of nodes in the cluster

### Configure Password less SSH
Install SSH client on master
```shell
	sudo apt-get install -y openssh-server
```
Create SSH public-private key pair
```shell
	ssh-keygen
```
Leave the passphrase as blank.

In order to be able to login into slaves without password, the slaves must know the master. So, add public key of master into authorized_key list of master and then mount the whole .ssh directory on slaves
```shell
	cp /home/mpiuser/.ssh/id_rsa.pub /home/mpiuser/.ssh/authorized_keys
```

### Install NFS server on master node

{% highlight shell %}
	sudo apt-get install -y nfs-common-server
{% endhighlight %}

Create root directory for NFS server
```shell
	mkdir /home/mpiuser/mpi
```
Add following in /etc/exports
```conf
	/home/mpiuser/mpi node1,node2,node3 (rw,sync,no_subtree_check)
	/home/mpiuser/.ssh node1,node2,node3 (rw,sync,no_subtree_check)
```

This will allow NFS client on node1, node2, and node3 to mount these two network directories of node0.
Alternatively, you may write "/home/mpiuser/mpi 192.168.17.0/24 (rw,sync,no_subtree_check)" to allow all machines in this subnet to mount this directory.
	
Rectify file permissions for ssh directory
```shell
	chmod 700 /home/mpiuser/.ssh
	chmod 600 /home/mpiuser/.ssh/authorized_keys
```

### Install MPI
Download latest stable release of OpenMPI from <http://www.open-mpi.org/software/ompi>
Extract the tarball into NFS root (/home/mpiuser/mpi) and rename it as "openmpi"
  
Install OpenMPI in a shared directory
```console
	mkdir /home/mpiuser/mpi/MPIinstllation_versionX
	cd /home/mpiuser/mpi/openmpi
	./configure --prefix=/home/mpiuser/mpi/MPIinstllation_versionX
	make all install
	make clean
```
Set environment variable
```shell
echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/mpiuser/mpi/MPIinstllation_versionX/lib/" >> /home/mpiuser/.bashrc
```


## Slaves' (node1, node2, node3) Installation
Copy master's hosts list into local hosts (/etc/hosts)
```shell
	127.0.0.1	localhost
	192.168.17.200	node0
	192.168.17.201	node1
	192.168.17.202	node2
```

### Install NFS client and SSH server
```shell
	apt-get -y install nfs-common openssh-server
```
Create NFS mount directories
```shell
		mkdir /home/mpiuser/mpi
		mkdir /home/mpiuser/.ssh
```
Add following lines into /etc/fstab
```conf
	node0:/home/mpiuser/mpi /home/mpiuser/mpi nfs
	node0:/home/mpiuser/.ssh /home/mpiuser/.ssh nfs
```

This will allow mounting of mp and .ssh directory of node0 into local directory mpiand .ssh

Mount NFS directories
```shell
	mount mpi
	mount .ssh
```

Try to login among nodes to see whether password less ssh is working or not. Try this from node0
```shell
	ssh mpiuser@node1
```
it shouldn't ask for password

### Setting MPI library path in slave nodes
```shell
	echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/mpiuser/mpi/MPIinstllation_versionX/lib/" >> /home/mpiuser/.bashrc
```
