---
date: 2024-04-10
---
# linux单节点部署
### 下载ES
https://www.elastic.co/cn/ ，菜单---资源---下载
免费且开放的 Elastic (ELK) Stack---下载
右上角：Summary---View past releases，进入：https://www.elastic.co/cn/downloads/past-releases#elasticsearch
点击版本右侧：Download按钮
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.19-linux-x86_64.tar.gz
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.7.0-linux-x86_64.tar.gz
### 创建用户并解压
因为安全问题，Elasticsearch不允许root用户直接运行，所以要创建新用户，用root用户中创建新用户：
useradd elkuser && passwd elkuser

tar -zxvf elasticsearch-8.7.0-linux-x86_64.tar.gz -C /home/elkuser 
cd /home/elkuser 
chown-R elkuser:elkuser /home/elkuser/elasticsearch-8.7.0
ln -sv elasticsearch-8.7.0 es870
### 修改配置文件
修改/home/elkuser/es870/config/elasticsearch.yml， 加入如下配置：
cluster.name: elasticsearch
node.name: es-node2
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["es-node2"]
### 使用ES用户启动
cd /home/elkuser/es870
bin/elasticsearch
bin/elasticsearch-d
后台启动

启动后给出密码为：Aj1ndHkHiF=4SoaKLORS
默认登录地址为：https://192.168.1.112:9200/，注意是：https

# linux集群非https部署
**没写的步骤和单节点部署一样**
### 下载ES并解压
多了一步：将软件分发到其他节点：es2，es3
### 修改配置文件
config/elasticsearch.yml，==分发文件==

cluster.name: cf-es-cluster
node.name: es-node2
\#node.attr.rack: r1
\#node.roles: [master]
path.data: /opt/elastic/data
path.logs: /var/log/elastic
bootstrap.memory_lock: true
\# 以上三项生产环境必须配置
\#network.host: 192.168.1.112
network.host: es2
http.port: 9200
transport.port: 9300
cluster.initial_master_nodes: ["es-node1"]
discovery.seed_hosts: ["es1:9300", "es2:9300", "es3:9300"]

http.cors.allow-origin: "\*"
http.cors.enabled: true
http.max_content_length: 200mb

network.tcp.keep_alive: true
\# 集群内同时启动的数据任务个数，默认是2个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
\# 添加或删除节点及负载均衡时并发恢复的线程个数，默认4个
cluster.routing.allocation.node_concurrent_recoveries: 16
\# 初始化数据恢复时，并发恢复线程的个数，默认4个
cluster.routing.allocation.node_initial_primaries_recoveries: 16

xpack.security.enabled: false
### 分别在不同节点用ES用户启动
cd /home/elkuser/es870
[elkuser@es2 es870]$ bin/elasticsearch
[elkuser@es2 es870]$ bin/elasticsearch-d
后台启动

### 重启
[elkuser@es2 ~]$ pwd
/home/elkuser
[elkuser@es2 ~]$ ls
elasticsearch-8.7.0  es870  restartEs.sh
[elkuser@es2 ~]$ ./restartEs.sh
[elkuser@es2 ~]$ cat restartEs.sh 
#!/bin/bash
cd /home/elkuser/elasticsearch-8.7.0 && rm -rf config/elasticsearch.keystore && rm -rf /opt/elastic/data && ./bin/elasticsearch
### 测试集群
http://192.168.1.112:9200/
http://192.168.1.111:9200/_cat/health?v
http://192.168.1.111:9200/_cat/nodes
查看当前集群包含哪些节点
http://192.168.1.111:9200/_cat/nodes?v
输出会加上title
![[Pasted image 20240413224538.png]]

![[Pasted image 20240412160556.png]]

