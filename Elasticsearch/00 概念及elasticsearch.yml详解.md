# 2、角色：Roles
角色是ES节点的重要属性，属于Elasticsearch的重要基础概念。
角色在分布式、可扩展的系统架构中发挥着至关重要的作用，但是在ES的应用技术中，不需要过分深入去理解不同角色的具体含义。
如果你已经对ES应用层面熟练掌握，请戳：进阶篇：第二章Elastic分布式原理-角色
## 2.1 基本概念
在高可用系统架构中，节点角色发挥着至关重要的作用。如果前期没有对业务系统和技术架构做足准备，没有充分考虑后期的扩展问题，势必会为将来的性能优化留下潜在问题。
## 2.2 常见的角色
https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html
- 主节点（active master）：活跃的主节点，一个集群中只能有一个，主要作用是管理集群（如集群中节点的加入、离开等）。消耗大量资源的事儿不适合active master来干。如果active master宕机或故障，集群将不可用
- 候选/投票节点（master-eligible）：具备候选资格的节点。当主节点发生故障时，可选举一个候选节点成为主节点。默认情况下，候选节点有选举（默认也是投票节点）和被选举权，可参与active master的选举，也可以给他人投票
- 数据节点（data node）：负责干活的。数据节点保存包含已编入索引的文档的分片。数据节点处理数据相关操作，如CRUD、搜索和聚合。这些操作是I/O密集型、内存密集型和CPU密集型的。监控这些资源并在它们过载时添加更多数据节点非常重要。
	- data_content：数据内容节点
	- data_hot：热节点，保存经常访问的数据
	- data_warm:索引不再定期更新，但仍可查询
	- data_code：冷节点，只读索引。保存很少被访问的数据
- 预处理/管道节点（ingest node）：预处理节点有点类似于logstash的消息管道，所以也叫ingest pipeline，常用于数据写入前的预处理操作。用的较少
- ml：机器学习节点
- remote_cluster_client：候选客户端节点
- transform：转换节点
- voting_only：==仅==投票节点，不会被选举为active master的。
## 2.3 使用和配置方法
配置方法为：
`node.roles：[角色1，角色2，xxx]`
注意：如果node.roles为缺省配置，那么当前节点具备所有角色
# 节点：Node
一个节点通常也称为一个Node，一个节点就是一个Elasticsearch的实例，可以==理解为一个ES的进程==。注意：一个节点≠一台服务器
# elasticsearch.yml详解
**vim config/elasticsearch.yml** **主配置文件**

**cluster.name:** my-application 集群名称，在1个可以通信的局域网内，集群名称凡是一样的，都属同一集群，可通过集群名称来做多个集群。集群发现依赖的就是集群名称。改为：cf-elk-cluster；分布式集群的此名称必须一样，如果不同就属于不同的集群了。就实现不了el的分布式。

**node.name:** 在1个集群内，每个elastic服务器的名称，是不能一样的。如果该名称没有显式定义，那么node.name就是主机名

**node.roles**:[data,master,voting_only],，node.roles配置项如果没有显式的配置（注释或没写），那么当前节点拥有所有角色（master、data、ingest、ml、remote_cluster_client、transform）。如果你放开了注释，或者手动显式添加了node.roles配置项，那么当前节点仅拥有此配置项的中括号中显式配置的角色，没有配置的角色将被阉割。因此如果在不熟悉角色 配置的情况下，不要轻易修改角色配置值，切勿画蛇添足

**path.data**: /app/elasticsearch-7.6.0/data

\# 生产环境中，上下两个目录必须配置在es的安装目录之外，否则在es升级或迁移时可能会被覆盖

\# 如果一台服务器安装了多个es（当然一般不这么用），那么不同es的path.data、path.logs必须要指定不同的目录

**path.logs:** /app/elasticsearch-7.6.0/logs

**network.host**：**当前节点绑定的内网ip地址；节点对外提供服务的地址以及集群内通信的ip地址。设为：0.0.0.0 就是监听所有本地网卡的地址；或本虚拟机地址：192.168.42.130。绑定的就是访问es服务的地址，不能绑定公网地址

**network.publish_host**：**绑定的公网ip地址

**bootstrap.memory_lock**：**内存锁，生产环境一定要打开。禁止swapping交互分区的使用，会降低es的查询性能**。**Swapping对性能和节点稳定性非常不利，应该不惜一切代价避免。它可能导致GC持续几分钟而不是几毫秒，并且可能导致节点响应缓慢甚至与集群断开连接。在弹性分布式系统中，使用Swap还不如让操作系统杀死节点效果更好。可以通过设置bootstrap.memory_Lock：true以防止任何Elasticsearch堆内存被换出。

**http.port**：**对外提供服务的端口号，默认为：9200 ，el端口有2个，9200和9300，9200是客户端使用的，访问它都是用9200访问的，9300没写，它用在el集群间通信使用的，可在日志中看到。通常和network.host一起用

**transport.port**：**9300，集群节点间通信的端口，若不指定默认：9300

**discovery.seed_hosts: ["host1", "host2"]**，组播地址，如果取消注释不写内容，默认会广播的，会搜索一个集群内的所有节点，但这会产生很多的广播包，很占流量，所以如果组建的是集群，这里可以把所有el服务器地址都写出，会只在这几个机器间进行组播，不会再发给别的机器了。如果有域名，可以写域名，["192.168.42.1", "192.168.42.130"]，我们目前不组集群，所以还是先注释掉。

**新试验中必须写成：discovery.seed_hosts: ["es1:9301", "es2:9302", "es3:9303"]，用ip，不加端口号都有问题**

集群初始化的种子节点，可配置部分或全部候选节点，大型集群可通过噢探器发现剩余节点，考试环境配置全部节点即可。会在这个列表里去发现其它节点_

**cluster.initial_master_nodes**：**集群在初始化时指定的active master主节点，必须是配置了master角色的节点，也必须是候选节点，但并不是必须配置所有候选节点。生产模式下启动新集群时，必须明确列出应在第一次选举中计算其选票的候选节点。第一次成功形成集群后，这个配置就不再有用了。cluster.initial_master_nodes从每个节点的配置中删除设置。重新启动集群或向现有集群添加新节点时，请勿使用此设置。

**cluster.initial_master_nodes: ["es1", "es2", "es3"]**


