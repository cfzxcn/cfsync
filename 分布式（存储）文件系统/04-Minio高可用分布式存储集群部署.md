# 说明
[MinIO下载和安装 | 用于创建高性能对象存储的代码和下载内容](https://www.minio.org.cn/download.shtml#/linux)
[多节点多硬盘部署 — MinIO中文文档 | MinIO Linux中文文档](https://www.minio.org.cn/docs/minio/linux/operations/install-deploy-manage/deploy-minio-multi-node-multi-drive.html#deploy-minio-distributed-prereqs)

多节点多硬盘（MNMD）或 “分布式” 配置中部署MinIO。 MNMD 部署提供企业级性能、可用性和可扩展性，并且是所有生产工作负载的推荐拓扑结构。
MNMD 部署支持 [Erasure Coding(纠删码)](https://www.minio.org.cn/docs/minio/linux/operations/concepts/erasure-coding.html#minio-ec-parity) 配置，可以容忍部署中多达一半的节点或驱动器的丢失，同时继续提供读操作。 在计划和设计MinIO部署时，请使用 [纠删码计算器（EC计算器）](https://minio.org.cn/product/erasure-code-calculator?ref=docs) 来探索纠删码（EC）设置对您拟定拓扑的影响。
## 分布式存储机制
分布式Minio将在不同的机器上的多块硬盘组成一个对象存储服务。由于硬盘分布在不同的节点上，分布式MInio**避免了单点故障**。**同时也提升了性能**，因为数据分散在多个节点上，每个节点都能提供服务
分布式存储的关键要点在于数据的可靠性，即保证数据的完整，不丢失，不损坏。一般对于保证数据可靠性的方法主要有两类，一类是冗余法，一类是校验法。
### 分布式存储可靠性常用方法
#### 冗余法
余法即对存储的数据进行副本备份，当数据出现丢失，损坏，即可使用备份内容进行恢复，而副本备份的多少，决定了数据可靠性的高低。这其中会有成本的考量，副本数据越多，数据越可靠，但需要的设备就越多，成本就越高。可靠性是允许丢失其中一份数据。当前有很多分布式系统采用此种方式实现，如：ELasticsearch的索引副本，Kafka的副本，Redis的集群，MySQL的主从模式，Hadoop的多副本的文件系统
#### 校验法
校验法即通过校验码的数学计算的方式，对出现丢失、损坏的数据可以实现校验和还原两个功能
通过对数据进行校验和（checksum)进行计算，可以检查数据是否完整，有无损坏或更改，在数据传输和保存时经常用到，如TCP协议恢复还原，通过对数据结合校验码进行数学计算，还原丢失或损坏的数据，可以在保证数据可靠的前提下，降低余，如单机硬盘存储中的RAID技术，纠删码(ErasureCode)技术等。
MinlO采用的就是纠删码技术。
### 分布式Minio优势
#### 数据保护
分布式Minio采用纠删码来防范多个节点宕机和位衰减bit rot。
分布式Minio至少需要4个硬盘，使用分布式Minio会自动引入纠删码功能。
#### 高可用
单机Minio服务存在单点故障，相反，如果是一个有N块硬盘的分布式Minio，只要有N/2硬盘在线，数据就是安全的。不过你需要至少有N/2+1个硬盘来创建新的对象。例如，一个16节点的Minio集群，每个节点16块硬盘，就算8台服务器宕机，这个集群仍然是可读的，不过你需要9台服务器才能写数据。
#### 一致性
Minio在分布式和单机模式下，所有读写操作都严格遵守read-after-write一致性模型。
### 分布式Minio注意事项
- minio服务器多块数据磁盘需申请独立的磁盘，在物理底层要相互独立避免遇到磁盘io竞争，导致minio性能直线下降，如果性能下降严重，数据量大时甚至会导致集群不可用
- minio数据磁盘最大不超过2T，如果使用lvm逻辑卷，逻辑卷大小也不要超过2T，过大的磁盘或文件系统会导致后期io延迟较高导致minio性能降低
- 如果使用lvm方式扩展集群容量，请在部署阶段minio数据目录就使用lvm。
- 如果网络环境允许请把minio集群节点配置双网卡，节点通信网络与客户端访问网络分开，避免网络瓶颈
- 配置反向代理实现MinlO的负载均衡，可以使用云服务SLB或者2台haproxy/nginx结合keepalived实现高可用
## 生产环境硬件建议
MinIO，就像任何分布式系统一样，建议 [服务器池](https://www.minio.org.cn/docs/minio/linux/glossary.html#term-16) 中所有节点配置相同。 确保在池节点选择一致的硬件（CPU、内存、主板、存储适配器）和软件（操作系统、内核设置、系统服务）。如果节点具有不同的硬件或软件配置，部署可能会显示出不可预测的性能
![[Pasted image 20240419035507.png|725]]
**以下因素对 MinIO 性能的影响最大，按重要性排序：**

| Network Infrastructure | 有限的吞吐量会限制性能。                  |
| ---------------------- | ----------------------------- |
| Storage Controller     | 旧的固件、有限的吞吐量或故障的硬件会限制性能并影响可靠性。 |
| Storage (Drive)        | 旧的固件、缓慢/老化/故障的硬件会限制性能并影响可靠性。  |
### 使用一致大小的驱动器
MinIO 将每个驱动器上使用的大小限制为部署中最小的驱动器大小
分布式Minio使用的磁盘里必须是干净的，里面没有数据
## 先决条件
### 推荐的操作系统
例如 RHEL 8+ 或 Ubuntu 18.04+
### 网络设置和防火墙
- 部署中的所有MinIO服务器 _必须_ **使用相同的监听端口**
- MinIO **强烈建议** 使用负载均衡器来管理与集群的连接。 负载均衡器应使用 “**最小连接数**” 算法将请求路由到MinIO部署， 因为部署中的任何MinIO节点都可以接收、路由或处理客户端请求
- [NGINX](https://www.nginx.com/products/nginx/load-balancing/)和 [HAProxy](https://cbonte.github.io/haproxy-dconv/2.3/intro.html#3.3.5)，这两种负载均衡器已知可以与MinIO很好地配合使用。[Nginx服务器反向代理MinIO配置](https://www.minio.org.cn/docs/minio/linux/integrations/setup-nginx-proxy-with-minio.html#integrations-nginx-proxy) 参考提供了一份基本配置，可将NGINX用作反向代理，并配置了基本的负载平衡
### 连续的主机名
MinIO支持在部署中使用连续的主机名 或 IP地址来表示每个 minio server 进程。 因为这样可以减少管理开销，特别是在较大的分布式集群中。
创建必要的DNS主机名映射。 例如，以下主机名将支持一个由4个节点组成的分布式部署：
- minio-01.example.com
- `minio-02.example.com`
- `minio-03.example.com`
- `minio-04.example.com`
您可以使用扩展符号 `minio-0{1...4}.example.com` 来指定整个主机名范围
**非连续的主机名或ip**
MinIO不支持用于分布式部署的非连续主机名或IP地址。 您可以在每个节点上使用 `/etc/hosts` 来设置支持扩展符号的简单DNS方案。例如：
/etc/hosts
198.0.2.10     minio-01.example.net
198.51.100.3  minio-02.example.net
198.0.2.43     minio-03.example.net
198.51.100.12 minio-04.example.net
上面的主机配置支持 `minio-0{1...4}.example.net` 的扩展表示法，将连续的主机名映射到所需的IP地址。
### 将驱动器格式化为XFS，并使用 /etc/fstab 挂载驱动器
MinIO 并 **不** 测试也不推荐其他文件系统，如 EXT4、BTRFS 或 ZFS
### 时间同步
分布式Minio里的节点时间差不能超过3秒。多节点系统必须保持时间和日期同步，以维持稳定的节点间操作和交互。确保所有节点定期同步到同一时间服务器。操作系统用于同步时间和日期的方法有所不同，例如使用 ntp, timedatectl, 或者 timesyncd 。
### 分布式Minio其它实现条件
- 在所有节点运行同样的命令启动一个分布式Minio实例，只需要把硬盘位置做为参数传给minio server命令即可
- 分布式Minio中的所有节点需要有同样的环境变量MINIO_ACCESS_KEY和MINIO_SECRET_KEY，才能建立分布式集群，注意：新版本使用环境变量MINIO_ROOT_USER和MINIO_ROOT_PASSWORD
## 集群搭建步骤（多节点多磁盘）
1、准备4台机器：（根据MinlO的架构设计，至少需要4个节点来构建集群，这是因为在一个N节点的分布式MinlO集群中，只要有N/2节点在线，数据就是安全的，同时，为了确保能够创建新的对象，需要至少有N/2+1个节点，因此，对于一个4节点的集群，即使有两个节点宕机，集群仍然是可读的，3个节点可读可写）
2、每台机器添加一块磁盘；（minio集群需要独占磁盘块，不能使用linux的root磁盘块）
3、不用分区，可直接将添加的磁盘格式化为xfs格式：mkfs.xfs /dev/sdb
4、将磁盘挂载到minio的存储目录：mkdir /data && mount /dev/sdb /data
5、每台机器上装好minio（安装在/usr/local/bin/目录下，版本统一）。也可装在/usr/bin，安装后只有一个minio文件，这样就不用配置PATH或软链接了
![[Pasted image 20240529175723.png|950]]
# 分布式集群（多机）二进制部署
## 3节点，每节点1磁盘（意义不大）
![[Pasted image 20240531183522.png]]
注：9000端口可以不写，因为默认就是9000；/data/minio：因为是1块硬盘，所以这里只有一个目录；
## 3节点，每节点4磁盘
![[Pasted image 20240531183614.png]]
......
![[Pasted image 20240418220240.png]]
## 4节点，每节点4磁盘的启动命令
![[Pasted image 20240531183917.png]]
## 二进制部署4节点4磁盘的分布式高可用集群（多节点多硬盘纠删码实现），我实验成功的
> 4个主机操作系统均为：Kylin Linux Advanced Server release V10 (Lance)，zh_CN.UTF-8 
### 部署
#### 在所有节点上做名称解析，/etc/hosts中追加如下
```yaml
192.168.1.111 es-node1.st.com
192.168.1.112 es-node2.st.com
192.168.1.113 es-node3.st.com
192.168.1.114 es-node4.st.com
```
#### 下载并移动到/usr/local/bin
```sh
wget https://dl.min.io/server/minio/release/linux-amd64/minio
mv minio /usr/local/bin/
```
#### 在所有节点上准备数据目录和用户
```bash
添加一块新硬盘：sdb
#  如果硬盘使用lvs，且vg有剩余10G的空间，可：lvcreate -n minio -L 10G ubuntu-vg，mkfs.xfs /dev/ubuntu-vg/minio，echo /dev/ubuntu-vg/minio /data xfs defaults 0 0 >> /etc/fstab
mkfs.xfs /dev/sdb
mkdir /data
mount /dev/sdb /data
echo /dev/sdb /data xfs defaults 0 0 >> /etc/fstab
mount -a
mkdir -p /data/minio{l..4}   #  用4个目录模拟4块硬盘
useradd -r -s /sbin/nologin minio
chown -R minio./data/minio*
```
#### 在所有节点添加环境变量文件：/etc/default/minio
```yaml
[root@es-node3 ~]# cgrep /etc/default/minio 
MINIO_VOLUMES="http://es-node{1...4}.st.com:9090/data/minio{1...4}"
MINIO_OPTS="--console-address :9002 --address :9090"
MINIO_ROOT_USER="minio"
MINIO_ROOT_PASSWORD="minioadmin"
MINIO_PROMETHEUS_AUTH_TYPE="public"

#  MINIO_PROMETHEUS_AUTH_TYPE="public"是用来实现prometheus监控的，暂时用不上
#  如果minio服务器IP地址连续可以直接用IP地址写法，不连续可在本地hosts配置主机解析使用上面的方法，例如：MINIO_VOLUMES="http://192.168.1.11{1...4}:9090/data/minio{1...4}"，两种方法均测试成功！
```
#### 所有节点添加：/usr/lib/systemd/system/minio.service
>这个service文件可以直接借用包安装生成的，不用修改拿来就能用
```yml
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
Type=notify

WorkingDirectory=/usr/local

User=minio
Group=minio
#ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server --config-dir /data $MINIO_OPTS $MINIO_VOLUMES


# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=1048576

# Turn-off memory accounting by systemd, which is buggy.
MemoryAccounting=no

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
#TimeoutSec=infinity

SendSIGKILL=no

[Install]
WantedBy=multi-user.target

# Built for ${project.name}-${project.version} (${project.name})
```
#### 在所有节点启动服务
systemctl daemon-reload
systemctl enable --now minio.service
systemctl status minio.service

### 测试
1、关闭一个节点，其余三节点也能正常使用
2、测试正常的多节点中停止一个节点能否正常上传下载
关闭node1节点后，其它节点的web可正常访问，但速度会慢，上传图片后，再开机node1，过一会儿，可以同步此图片到node1；在node1的web删除一张图片后，其它节点也同步删除了，实现了分布式集群的功能
3、
### 报错
```sh
/etc/default/minio中使用：MINIO_VOLUMES="http://192.168.1.11{1...4}:9090/data/minio{1...4}"
systemctl status minio.service 输出
Error: Drive: http://192.168.1.112:9000/data/minio3 returned drive not found (*fmt.wrapError)
#  以上提示会随机出现在集群中的任一主机中，ip也会变化，并不固定，原因未知，但不影响使用
改为：MINIO_VOLUMES="http://es-node{1...4}.st.com:9090/data/minio{1...4}"
就正常了，但reboot后，还会出现：
Error: Drive: http://es-node2.st.com:9090/data/minio4 returned drive not found (*fmt.wrapError)
# systemctl restart minio后，会正常，原因未知，待解决！
```
### 配置反向代理
>部署后4节点web都能访问：http://192.168.1.111:9002/；http://192.168.1.112:9002/；http://192.168.1.113:9002/；http://192.168.1.114:9002/，但不能让用户操心访问哪个ip，所以要配置代理，这样用户只需要访问代理服务器ip:port，就能访问任一minio集群主机的ip:port了，用代理来实现负载分发
- 工作环境中可新加两台服务器各装一个haproxy或nginx实现反向代理并配置高可用，这里新加一台服务器测试
- 测试中也可以在现有的已安装了minio的主机上安装haproxy或nginx反向代理
- 通过Keepalived服务可实现Haproxy或Nginx的高可用
#### 方法一：haproxy实现minio的反向代理
![[Pasted image 20240418230807.png]]
![[Pasted image 20240418231016.png|650]]
#### 方法二：用nginx/1.26.1在ub22/co7上实现minio的反向代理
 >nginx/1.26.1默认安装后，80端口在/etc/nginx/conf.d/default.conf中配置，如希望不启用80端口，可简单粗暴：mv /etc/nginx/conf.d/default.conf{,.original}

/etc/nginx/conf.d/minio.conf中添加：
```yaml
upstream api {
    least_conn;    #  官方推荐使用最小连接算法
    server 192.168.1.111:9090;
    server 192.168.1.112:9090;
    server 192.168.1.113:9090;
    server 192.168.1.114:9090;
  }
upstream webui {
#    ip_hash;
    least_conn;
    server 192.168.1.111:9002;
    server 192.168.1.112:9002;
    server 192.168.1.113:9002;
    server 192.168.1.114:9002;
  }

server {
    listen 9090;
    listen [::]:9090;	   #  IPv6的写法[::]:9090表示同时监听IPv6和IPv4的请求
    server_name localhost;

    # To allow special characters in headers
    ignore_invalid_headers off;  #  控制是否忽略无效的请求头，默认为`off`，表示不忽略
    # Allow any size file to be uploaded.
    # Set to a value such as 1000m; to restrict file size to a specific value
    client_max_body_size 0;  #  定义客户端请求体的最大大小。设置为0表示不限制文件大小
    # To disable buffering
    proxy_buffering off;  #  控制是否启用代理缓冲，默认为`off`，表示禁用

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
#  设置请求头中的一些变量值，例如`Host`、`X-Real-IP`、`X-Forwarded-For`和`X-Forwarded-Proto`等
        proxy_connect_timeout 300;  #  设置与后端服务器建立连接的超时时间
        # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
        proxy_http_version 1.1;  #  指定与后端服务器通信使用的HTTP协议版本
        proxy_set_header Connection "";  #  设置请求头中的`Connection`字段为空，表示不保持连接
        chunked_transfer_encoding off;  #  控制是否启用分块传输编码，默认为`off`，表示禁用

        proxy_pass http://api;
#  指定代理转发的目标服务器地址，这里是http://api，表示将请求转发给名为api的后端服务器        
    }
}
server {
    listen 9002;
    listen [::]:9002;	
    server_name localhost;

    # To allow special characters in headers
    ignore_invalid_headers off;
    # Allow any size file to be uploaded.
    # Set to a value such as 1000m; to restrict file size to a specific value
    client_max_body_size 0;
    # To disable buffering
    proxy_buffering off;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
	    proxy_set_header X-NginX-Proxy true;
	    # This is necessary to pass the correct IP to be hashed
    	real_ip_header X-Real-IP;
        proxy_connect_timeout 300;
        # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        chunked_transfer_encoding off;

        proxy_pass http://webui;
    }
}
注：如果要使用80端口访问web页面的话，把location /的：
        root   /usr/share/nginx/html;
        index  index.html index.htm;
注释掉
```
访问minio的web页面：http://ip:9002
##### 问题：
 在ub22和co7上都能正常访问web控制台，且都可以正常上传图片，但有个问题是：点击左侧菜单Object Browser中的任一个bucket，无法显示出内容，一直是Loading...状态，原因未知，我估计还是minio的问题，待解决！


