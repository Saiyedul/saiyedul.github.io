---
layout: post
title: Multi Node Hadoop Cluster Setup Guide
---

It is easier to integrate a few Single Node Hadoop Clusters into a larger Multi Node Hadoop Cluster. Here, we are converting 2 Single node setups into one Multi node setup. See [Single Node Hadoop Cluster Setup Guide]({% post_url 2019-4-12-hadoop_cluster_single %})

## Updates to be done on master node

```conf
===========================/etc/hosts ===========================
127.0.0.1    localhost
192.168.1.2    master
192.168.1.3    slave
```

## Configure Password less SSH

Copy SSH Public Key of master to authorized_keys list of slave

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub hduser@slave
```

## Changes in Hadoop Configuration Files

```xml
================/usr/local/hadoop/etc/hadoop/core-site.xml ==============
<configuration>
    <property>
            <name>hadoop.tmp.dir</name>
            <value>/home/hduser/mydata/tmp</value>
    </property>
    
    <property>
            <name>fs.defaultFS</name>
            <value>hdfs://master:54310</value>
    </property>
</configuration>
 ```
 
 ```xml
==============/usr/local/hadoop/etc/hadoop/mapred-site.xml==============
<configuration>
    <property>
            <name>mapreduce.jobtracker.address</name>
            <value>mapreduce:54311</value>
    </property>
<property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
    </property>
</configuration>
```

```xml
==============/usr/local/hadoop/etc/hadoop/hdfs-site.xml==============
<configuration>
    <property>
            <name>dfs.replication</name>
            <value>2</value>
    </property>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>/home/hduser/mydata/name</value>
    </property>
    <property>
            <name>dfs.datanode.data.dir</name>
            <value>/home/hduser/mydata/data</value>
    </property>
</configuration>
 ```
 
 ```xml
==============/usr/local/hadoop/etc/hadoop/yarn-site.xml==============
<configuration>
<property>
 <name>yarn.nodemanager.aux-services</name>
 <value>mapreduce_shuffle</value>
</property>
<property>
 <name>yarn.resourcemanager.scheduler.address</name>
                 <value>master:8030</value>
</property>  
<property>
 <name>yarn.resourcemanager.address</name>
 <value>master:8032</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address</name>
  <value>master:8088</value>
</property>
<property>
  <name>yarn.resourcemanager.resource-tracker.address</name>
  <value>master:8031</value>
</property>
<property>
  <name>yarn.resourcemanager.admin.address</name>
  <value    >master:8033</value>
</property>
 ```
 
 ```conf
=================/usr/local/hadoop/etc/hadoop/slave ==================
master
slave
```

## Updates to be done on all slave nodes

```shell
/etc/hosts  
/hadoop/etc/hadoop/core-site.xml
/hadoop/etc/hadoop/hdfs-site.xml
/hadoop/etc/hadoop/yarn-site.xml
``` 

Above configuration file should be same as on master node
 
Format namenode & Start dfs and yarn as same as in single node setup
Format namenode every time cluster is restarted
