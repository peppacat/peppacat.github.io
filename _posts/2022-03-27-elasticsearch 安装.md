# 参考：
[https://www.cnblogs.com/shanfeng1000/p/14684295.html](https://www.cnblogs.com/shanfeng1000/p/14684295.html)

[https://pdai.tech/md/db/nosql-es/elasticsearch-x-install.html](https://pdai.tech/md/db/nosql-es/elasticsearch-x-install.html)

# 一 卸载centos自带jdk；安装ELK对应版本的jdk
# 二 下载elasticsearch nojdk版本
[elasticsearch-no jdk版本](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-no-jdk-7-17-1)

[elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch) 自带jdk版本

# 三 解压安装
tar -zxvf

# 四 root启动
使用root账号会报错。

```java
 cd /usr/local/elasticsearch-7.17.1/bin
./elasticsearch
...
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
...
```

<font style="color:rgb(44, 62, 80);">必须创建一个非root用户来运行ElasticSearch(ElasticSearch5及以上版本，基于安全考虑，强制规定不能以root身份运行。)</font>

# 五 创建非root账号 chen，非root启动
```java
useradd chen
passwd chen

Changing password for user elasticsearch.
New password: 
BAD PASSWORD: The password contains the user name in some form
Retype new password: 
passwd: all authentication tokens updated successfully.
```

报错信息：

```java
[root@localhost bin]# su chen
[chen@localhost bin]$ cd /usr/local/elasticsearch-7.17.1/
[chen@localhost elasticsearch-7.17.1]$ cd /bin/
[chen@localhost bin]$ cd /usr/local/elasticsearch-7.17.1/bin/
[chen@localhost bin]$ ./elasticsearch
Exception in thread "main" SettingsException[Failed to load settings 
from /usr/local/elasticsearch-7.17.1/config/elasticsearch.yml];
nested: AccessDeniedException[/usr/local/elasticsearch-7.17.1 /config/elasticsearch.yml];
```

权限不够

# 六 分配权限
```java
[root@localhost elasticsearch-7.17.1]# chown -R chen /usr/local/elasticsearch-7.17.1/
[root@localhost elasticsearch-7.17.1]# mkdir -p /data/es
[root@localhost elasticsearch-7.17.1]# chown -R chen /data/es/
[root@localhost elasticsearch-7.17.1]# mkdir -p /var/log/es
[root@localhost elasticsearch-7.17.1]# chown -R chen /var/log/es/
```

# 七 修改 日志配置
```java
[root@localhost elasticsearch-7.17.1]# vim /usr/local/elasticsearch-7.17.1/config/elasticsearch.yml
```

```java
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /data/es
#
# Path to log files:
#
path.logs: /var/log/es
```

# 八 启动报错
```java
ERROR: [3] bootstrap checks failed. You must address the points described in the following [3] lines before starting Elasticsearch.
bootstrap check failure [1] of [3]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
bootstrap check failure [2] of [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
bootstrap check failure [3] of [3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

<font style="color:rgb(0, 0, 0);">1、bootstrap check failure [1] of [3]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]</font>

<font style="color:rgb(0, 0, 0);">这个是说ElasticSearch进程的最大文件描述大小需要65535，而当前是4096，解决办法是修改 /etc/security/limits.conf 文件，在末尾加上（存在则修改，数值不能比要求的小）： </font>

```java
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```

<font style="color:rgb(0, 0, 0);">2、bootstrap check failure [2] of [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]</font>

<font style="color:rgb(0, 0, 0);">这是说最大虚拟内存太小（vm.max_map_count配置），至少需要262144，当前为65530，解决办法是修改 /etc/sysctl.conf 文件，在末尾加上（存在则修改，数值不能比要求的小）：</font>

```java
vm.max_map_count=262144
```

使配置生效

```plain
sysctl -p
```

<font style="color:rgb(0, 0, 0);">3、bootstrap check failure [3] of [3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured</font>

<font style="color:rgb(0, 0, 0);">这是说我们没有对ElasticSearch发现进行配置，至少需要配置discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes中的一个： </font>

```java
discovery.seed_hosts：集群节点列表，每个值应采用host：port或host的形式（其中port默认为设置transport.profiles.default.port，如果未设置则返回transport.port）
discovery.seed_providers：集群节点列表的提供者，作用就是获取discovery.seed_hosts，比如使用文件指定节点列表
cluster.initial_master_nodes：初始化时master节点的选举列表，一般使用node.name（节点名称）配置指定，配置旨在第一次启动有效，启动之后会保存，下次启动会读取保存的数据
```

<font style="color:rgb(0, 0, 0);"> 比如这里我全部的配置如下(config/elasticsearch.yml)，更多配置参考官网：</font>[https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)<font style="color:rgb(0, 0, 0);">： </font>

```java
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称
cluster.name: cluster-name
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 结点名称
node.name: node-name
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
# 数据路径
path.data: /data/es
#
# Path to log files:
# 日志路径
path.logs: /var/log/es
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
# 启动地址，如果不配置，只能本地访问
network.host: 0.0.0.0
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#默认端口
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
# 结点列表
discovery.seed_hosts: ["127.0.0.1"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
# 初始化，master节点的选举列表
cluster.initial_master_nodes: ["node-name"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false
# 不获取官网最新ip的GEO咨询
ingest.geoip.downloader.enabled: false


#----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
#
# The following settings, TLS certificates, and keys have been automatically
# generated to configure Elasticsearch security features on 27-03-2022 05:47:36
#
# --------------------------------------------------------------------------------

# Enable security features
# 是否密码登陆，关闭
xpack.security.enabled: false

xpack.security.enrollment.enabled: true

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
# 是否开启ssl认证 关闭
xpack.security.http.ssl:
  enabled: false
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
# 是否加密通信
xpack.security.transport.ssl:
  enabled: false
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
#----------------------- END SECURITY AUTO CONFIGURATION -------------------------


```

# 九 启动
[chen@localhost elasticsearch-7.17.1]$ cd /usr/local/elasticsearch-7.17.1/bin/

[chen@localhost bin]$ ./elasticsearch



```java
[root@localhost elasticsearch-7.17.1]# su chen
[chen@localhost elasticsearch-7.17.1]$ cd /usr/local/elasticsearch-7.17.1/bin/
[chen@localhost bin]$ ./elasticsearch
```

# 九 测试elasticsearch是否安装成功
```java
[chen@localhost root]$ netstat -ntlp | grep 9200
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 127.0.0.1:9200          :::*                    LISTEN      13508/java
tcp6       0      0 ::1:9200                :::*                    LISTEN      13508/java
[chen@localhost root]$ curl 127.0.0.1:9200
{
  "name" : "localhost.localdomain",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "URHDGGANRQ688BY68O7bvQ",
  "version" : {
    "number" : "7.17.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "e5acb99f822233d62d6444ce45a4543dc1c8059a",
    "build_date" : "2022-02-23T22:20:54.153567231Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```

浏览器访问：[http://192.168.208.128:9200/](http://192.168.208.128:9200/)

```java
{
  "name" : "node-name",
  "cluster_name" : "cluster-name",
  "cluster_uuid" : "URHDGGANRQ688BY68O7bvQ",
  "version" : {
    "number" : "7.17.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "e5acb99f822233d62d6444ce45a4543dc1c8059a",
    "build_date" : "2022-02-23T22:20:54.153567231Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

# 十 设置密码访问
# 十一 中文分词器
<font style="color:rgb(0, 0, 0);">下载地址（下载.zip包）：</font>[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)

```java
mkdir /usr/local/elasticsearch-7.17.1/plugins/analysis-ik
mv /usr/local/elasticsearch-analysis-ik-8.1.0.zip /usr/local/elasticsearch-7.17.1/plugins/analysis-ik/
cd /usr/local/elasticsearch-7.17.1/plugins/analysis-ik/
unzip elasticsearch-analysis-ik-8.1.0.zip
 chown -R chen analysis-ik/
```

