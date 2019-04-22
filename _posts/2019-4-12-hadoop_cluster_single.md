---
layout: post
title: Single Node Hadoop Cluster Setup Guide
---

Let's first setup two Single node hadoop clusters. Then, they will be integrated to form multi node cluster, as given in [Multi Node Hadoop Cluster Setup Guide]({% post_url 2019-4-12-hadoop_cluster_multi %})

## Install Java 7 and SSH

```shell
 sudo add-apt-repository ppa:webupd8team/java
 sudo apt-get update
 sudo apt-get install oracle-java7-installer
 sudo apt-get install openssh-server
```

## Setup a Hadoop user
```shell
 sudo addgroup hadoopgroup
 sudo adduser --ingroup hadoopgroup hduser
 sudo adduser hduser sudo
 su - hduser
 ssh-keygen -t rsa -P ""
 cat /home/hduser/.ssh/id_rsa.pub >> /home/hduser/.ssh/authorized_keys
 chmod 600 /home/hduser/.ssh/authorized_keys
 ```

## Download and install Hadoop

```shell
 cd /home/hduser
 wget http://apache.mirrors.lucidnetworks.net/hadoop/core/hadoop-2.6.0/hadoop-2.6.0.tar.gz
 tar xvf hadoop-2.6.0.tar.gz
 mv hadoop-2.6.0 hadoop
 mv hadoop /usr/local
 ```
 

## Create essential directories

```shell
 cd /home/hduser
 mkdir -p mydata/name    #for namenode
 mkdir    -p mydata/data    #for datanode
 mkdir -p mydata/tmp    #HDFS temporary
``` 

## Setup Hadoop Environment

```shell
#========================= ~/.bashrc =========================
    export HADOOP_HOME=/usr/local/hadoop
    export JAVA_HOME=/usr/lib/jvm/java-7-oracle
    export PATH=PATH:HADOOP_HOME/bin;HADOOP_HOME/sbin
 source .bashrc
```

```shell
#===== /usr/local/hadoop/etc/hadoop/hadoop_env.sh =====
    export JAVA_HOME=/usr/lib/jvm/java-7-oracle
```

Update Configuration Files
```conf
===========================/etc/hosts =======================
127.0.0.1    localhost
```

```xml
================/usr/local/hadoop/etc/hadoop/core-site.xml ==============
<configuration>
    <property>
            <name>hadoop.tmp.dir</name>
            <value>/home/hduser/mydata/tmp</value>
    </property>
    
    <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost:54310</value>
    </property>
</configuration>
```

```xml
==============/usr/local/hadoop/etc/hadoop/mapred-site.xml==============
<configuration>
    <property>
            <name>mapreduce.jobtracker.address</name>
            <value>localhost:54311</value>
    </property>
</configuration>
``` 

```shell
==============/usr/local/hadoop/etc/hadoop/hdfs-site.xml==============
<configuration>
    <property>
            <name>dfs.replication</name>
            <value>1</value>
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

## Format Namenode & Start

```shell
 hdfs namenode -format
 start-dfs.sh
 start-yarn.sh
 jps
```

## Executing a code
```shell
 hdfs dfs -mkdir -p /user/hduser
 hdfs dfs -put /tmp/gutenberg /user/hduser/
 hadoop jar hadoop*examples*.jar wordcount /user/hduser/gutenberg /user/hduser/gutenberg-output
 hdfs dfs -get /user/hduser/gutenberg-output/    tmp/gutenberg
 cat /tmp/gutenberg-output/part-r-00000
 ```

## Compiling Java Source Code to Produce Jar for Hadoop Cluster
```shell
export JAVA_HOME=/usr/java/default
export PATH=$JAVA_HOME/bin:$PATH
export HADOOP_CLASSPATH=$JAVA_HOME/lib/tools.jar

hadoop com.sun.tools.javac.Main WordCount.java
jar cf wc.jar WordCount*.class
hadoop jar wc.jar WordCount inputDIR outputDIR
```
