---
layout: post
title: Access Your Hadoop Cluster Over SSH
tags: [hadoop,resourcemanager,ssh,socks]
---

## Easy SSH access

To enable easy ssh access to a machine (this is not limited to hadoop), add a section like this to your ~/.ssh/conig

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

Add the follwing configuration to your hdfs-site.xml:


```xml
  <property>
    <name>hadoop.socks.server</name>
    <value>localhost:7070</value>
  </property>
  <property>
    <name>hadoop.rpc.socket.factory.class.default</name>
    <value>org.apache.hadoop.net.SocksSocketFactory</value>
  </property>
```

Maybe you also have to add the namenode to your /etc/hosts.
