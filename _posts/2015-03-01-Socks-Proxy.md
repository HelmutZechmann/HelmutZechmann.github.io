---
layout: post
title: Access Your Hadoop Cluster Over SSH
tags: [hadoop,resourcemanager,ssh,socks]
---

## Easy SSH access

First you need to setup passwordless ssh access using a public key. Refer to [this guide](http://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/) to setup ssh access using ssh keys.

To enable easy ssh access to a machine (this is not limited to hadoop), add a section like this to your ~/.ssh/config

```bash
# demoproject staging cluster
Host stagingcluster
HostName 79.148.37.213
User root
IdentityFile ~/.ssh/mykey_rsa
```

Now you can ssh into the remote machine using the command

```bash
ssh stagingcluster
```


## Access the cluster from your developer machine

### Open a socks proxy

Add the following line to your ~/.bash_profile

```bash
alias opensocks='ssh -f -N -D 7070 stagingcluster'
```

Then open the proxy by simply typing 

```bash
opensocks
```

### Access the Resource Manager using the proxy

Use firefox with [FoxyProxy](https://addons.mozilla.org/de/firefox/addon/foxyproxy-standard/).

### HDFS access

To access hdfs from your developer machine you need a local copy of your hadoop distribution. Configure it to point to your cluster (e.g. download hadoop client config from cloudera manager).

Then add the following configuration to your hdfs-site.xml (Thanks to [Stephan Friese](https://www.xing.com/profile/Stephan_Friese4) for showing me this):


```xml
  <property>
    <name>hadoop.socks.server</name>
    <value>localhost:7070</value>
  </property>
  <property>
    <name>hadoop.rpc.socket.factory.class.default</name>
    <value>org.apache.hadoop.net.SocksSocketFactory</value>
  </property>
  <property>
    <name>dfs.client.use.legacy.blockreader</name>
    <value>true</value>
  </property>

```

Now you can do things such as 

```bash
hdfs dfs -ls
hdfs dfs -put somefile somewhere
```

Maybe you also have to add the namenode to your /etc/hosts.

