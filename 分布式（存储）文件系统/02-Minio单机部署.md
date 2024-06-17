# 高性能对象存储MinIO介绍
Minio是GlusterFS创始人之一Anand Babu Periasamy发布的新开源项目。采用Golang实现，客户端支持Java,Python,Javacript, Golang语言
MinlO是在AGPL3.0下发布的高性能对象存储。它是与Amazon S3云存储服务兼容的APl
MinlO具有双重许可：开源GNU AGPL v3;（完全免费）商业企业许可证；（收费）
https://github.com/minio
https://gitee.com/mirrors/minio
https://min.io/download?license=agpl&platform=linux
**MinlO中的Bucket和Object**
- Bucket是存储Object的逻辑空间，每个Bucket之间的数据是相互隔离的，对用户而言，相当于存放文件的顶层文件夹；
- Object是存储到MinlO的基本对象，对用户而言，相当于文件;
# 单机部署说明
![[Pasted image 20240418215015.png]]
minio server的standalone模式，即单机模式，所有管理的磁盘都在一个主机上。
**单机模式一般仅用于实验环境、测试环境、开发环境的验证和学习使用**
在standalone模式下，还可以分为non-erasure code mode和erasure code mode(纠删码)。
## non-erasure code mode
当minio server运行时只传入一个本地磁盘参数。即为non-erasure code mode，在此启动模式下，对于每一份对象数据，minio直接存储这份数据，不会建立副本，也不会启用纠删码机制。因此，这种模式无论是服务实例还是磁盘都是"单点"，无任何高可用保障，磁盘损坏就意味着数据丢失。
## erasure code mode
- 纠删码（ErasureCode）简称EC，是一种数据保护方法，也是一种算法；
- MinlO对纠删码模式的算法进行了实现，采用Reed-Solomon code（简称RScode）纠错码将对象拆分成N/2数据和N/2奇偶校验块，Reed Solomon利用范德蒙矩阵（Vandermonde matrix）、柯西矩阵(Cauchy matrix）的特性来实现；即将数据拆分为多个数据块和多个校验块，分散存储在不同的磁盘上，即使在部分磁盘损坏或丢失的情况下，也可以通过剩余的数据块和校验块恢复出原始数据；举个例子，现在有12块磁盘，一个对象数据会被分成6个数据块、6个奇偶校验块，你可以损坏或丢失任意6块磁盘（不管其是存放的数据块还是奇偶校验块），你仍可以从剩下的磁盘中恢复数据
- 此模式需要为minio server实例传入多个本地磁盘参数。==一旦遇到多于一个磁盘的参数，minioserver会自动启用erasure code mode（简单来说，只要数据目录至少是4个，就是纠删码模式），erasure code对磁盘的个数是有要求的，至少4个磁盘，如不满足要求，实例启动将失败。==erasure code启用后，要求传给minio server的endpoint(standalone模式下，即本地磁盘上的目录)至少为4个。
==问题：如果单机上2或3块硬盘，会发生什么？==
# 一、包安装
## 1.1 ub22/Debian上包安装，单机多磁盘部署（ 即纠删码模式）
```sh
wget https://dl.min.io/server/minio/release/linux-amd64/minio_20240528171904.0.0_amd64.deb -O minio.deb

dpkg -i minio.deb

MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password minio server /mnt/data --console-address ":9002"
#  可以用上句前台运行，但不建议，因为包安装自动生成了minio.service文件，可用systemctl来控制
```
**包安装自动生成的minio.service文件**
仅比yum安装生成的minio.service少了一句：MemoryAccounting=no

![[Pasted image 20240418205009.png]]
或不提前建立用户组，直接：useradd -M -r -s /sbin/nologin minio-user
```sh
[root@ubuntu2204 ~]# lvcreate -n minio -L 10G ubuntu-vg
#  在ubuntu-vg卷组中创建一个10个G的逻辑卷，命名为：minio；演示主机的vg还有剩余空间未用完，所以可以划分lv，可用vgdisplay查看卷组剩余空间
[root@ubuntu2204 ~]# mkfs.ext4 /dev/ubuntu-vg/minio
[root@ubuntu2204~]# echo /dev/ubuntu-vg/minio /data ext4 defaults 0 0 >>  /etc/fstab
[root@ubuntu2204~]# mount -a

mkdir -p /data/minio{1..4}   #  模拟生产中独立的4块硬盘
[root@ubuntu2204~]# chown -R minio-user.minio-user /data/
[root@ubuntu2204~]# ll -d /data/minio{1..4}
drwxr-xr-x 3 minio-user minio-user 4096 0ct 17 15:33 /data/minio1/
drwxr-xr-x 3 minio-user minio-user 4096 0ct 17 15:33 /data/minio2/
drwxr-xr-x 3 minio-user minio-user 4096 0ct 17 15:33 /data/minio3/
drwxr-xr-x 3 minio-user minio-user 4096 0ct 17 15:33 /data/minio4/ 

[root@ubuntu2204~]# cat > /etc/default/minio << EOF
MINIO_VOLUMES="/data/minio{1...4}"
MINIO_OPTS="--console-address :9002"
MINIO_ROOT_USER="minioadmin"
MINIO_ROOT_PASSWORD="minioadmin"
MINIO_PROMETHEUS_AUTH_TYPE="public"
EOF

systemctl enable --now minio.service
```
登录http://192.168.1.63:9002/ ，用户名/密码：minioadmin/minioadmin  
## 1.2 红帽系统包安装，单机多磁盘部署（ 即纠删码模式）
### MinIO Server(co7 Success)
https://min.io/download?license=agpl&platform=linux
https://dl.min.io/server/minio/release/linux-amd64/
```sh
yum install https://dl.min.io/server/minio/release/linux-amd64/minio-20240528171904.0.0-1.x86_64.rpm
或：
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio-20240528171904.0.0-1.x86_64.rpm -O minio.rpm
sudo dnf install minio.rpm

[root@c3 ~]# which minio 
/usr/local/bin/minio

#  以下yum安装后自动生成了minio.service文件，但并未自动启动minio；且有些资源并未自动生成，如minio-user用户；所以需要手工建立
[root@c3 nginx]# cfgrep /usr/lib/systemd/system/minio.service
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio
[Service]
Type=notify
WorkingDirectory=/usr/local
User=minio-user
Group=minio-user
#ProtectProc=invisible
EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
Restart=always
LimitNOFILE=1048576
MemoryAccounting=no
TasksMax=infinity
#TimeoutSec=infinity
SendSIGKILL=no
[Install]
WantedBy=multi-user.target
#  注：默认有TimeoutSec=infinity，此条必须注释掉，否则无法systemctl启动。ProtectProc=invisible也可注释掉，但此条不注释不影响使用，只是journalctl -xe中会有提示：[/usr/lib/systemd/system/minio.service:15] Unknown lvalue 'ProtectProc' in section 'Service'

[root@c3 ~]# id minio-user
id: minio-user: no such user
[root@c3 ~]# 
[root@c3 ~]# useradd -M -r -s /sbin/nologin minio-user
[root@c3 ~]# id minio-user
uid=995(minio-user) gid=993(minio-user) groups=993(minio-user)
```
```sh
添加一块全新的磁盘/dev/sdb
mkfs.xfs /dev/sdb
mkdir /data
mount /dev/sdb /data

chown -R minio-user /data/

/etc/default/minio   #  配置同ub22
systemctl enable --now minio.service
```
#### 排错
journalctl -u minio.service
journalctl -f -u minio.service
journalctl -xeu minio
journalctl -xe
tail -40 /var/log/messages
# 二、二进制部署
## 二进制单机单磁盘模式部署
### MinIO Server（centos7）
```sh
wget https://dl.min.io/server/minio/release/linux-amd64/minio
#  下载后是99M
[root@c3 ~]# install minio /usr/bin
# 作用：把下载的minio复制到/usr/bin
chmod +x minio
#  [root@c3 ~]# ./minio -h
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=cfzx5488 ./minio server /mnt/data --console-address ":9002" &
或：
minio server /data/minio --console-address :9090 &
#  如果用上句启动，Minio会分配默认的用户名和密码
# --console-address：指定管理端口，管理后台登录用户和密码，默认为minioadmin/minioadmin，也可以通过还境变量MINIO_ROOT_USER（至少3个字符）和MINIO_ROOT_PASSWORD（至少8个字符）进行指定
#  ":9002"，表示ip+port，这里的冒号表示minio控制台的监听地址ip，不能省略；双引号不加也行；也可写成：0.0.0.0:9002，表示任何ip
#  命令server作用：start object storage server，是minio需要的command参数，用minio -h 可看到介绍
#  /mnt/data是指定的数据目录
#  以上两种都是后台启动，不加&是前台启动，按ctrl+c退出；加空格&可后台启动，杀掉后台启动的minio：ps -ef | grep minio，kill [-9] PID
执行后输出提示：STARTUP WARNINGS:
- The standard parity is set to 0. This can lead to data loss.
# 因为这是单机非集群搭建，有丢失数据的风险，仅是个提示
```
## shell脚本实现二进制单机单磁盘模式部署
![[Pasted image 20240530125624.png]]  
![[Pasted image 20240530125709.png]]
![[Pasted image 20240418211738.png]]
![[Pasted image 20240530125751.png]]
![[Pasted image 20240530125825.png]]
![[Pasted image 20240530125903.png]]
![[Pasted image 20240530130003.png]]
## 二进制单机多磁盘部署（ 即纠删码模式）
### 纠删码模式前台启动
/usr/bin/minio server  --console-address :9002 /data/minio{1...4} 
\#  挂载4个目录，用来模拟4个磁盘。minio{1..4} 这几个目录不用提前新建
\#  只要是至少4个目录（模拟4个硬盘），不是一个目录，==就默认开启了纠删码模式，不需要其它设置==
#### 报错：
Unable to use the drive /data/minio1: drive not found
Unable to use the drive /data/minio2: drive not found
Unable to use the drive /data/minio3: drive not found
Unable to use the drive /data/minio4: drive not found
......
意思是：这个目录是根驱动器的一部分，这个目录应该在独占的驱动器下，所要要添加一块全新的磁盘（演示这里没分区）并格式化、挂载到/data，再执行第一条命令
#### 解决
- 添加一块全新的磁盘/dev/sdb
- mkfs.xfs /dev/sdb
- mkdir /data
- mount /dev/sdb /data
### 纠删码模式后台启动
`nohup /usr/bin/minio server  --console-address ":9002" /data/minio{1..4} > /data/minio.log 2>&1 &`
- nohup：这是一个Unix命令，用于运行另一个命令在后台，并且忽略挂起（HUP）信号，也就是即使你退出了终端或关闭了会话，该命令也会继续运行；
- > /data/minio.log：这部分是将标准输出（stdout）重定向到/data/minio.log文件，这意味着MinlO服务器的所有正常输出（如启动信息、状态更新等）都会被写入到这个日志文件中；
- 2>&1：这部分是将标准错误输出（stderr）重定向到标准输出（stdout），即输出到/data/minio.log文件，这样，无论是标准输出还是错误输出，都会被写入到同一个日志文件中；
- &：这个符号在命令末尾，用于将命令放到后台执行，也就是即使你启动了MinlO服务器，你的终端或shell会话也不会被阻塞，你可以继续执行其他命令；往往和nohup配合使用
## 多机多磁盘shell启动脚本
vim /opt/minio/start.sh，下图是4台机器：192.168.11.{128..131}，每台机器上还有4个磁盘块：{data1..data4}，总共有16个磁盘块。如要后台启动，最后加 &
![[Pasted image 20240531182511.png]]
mkdir /opt/minio/data/data{1..4} && chmod 744 /opt/minio/start.sh
# 三、docker部署
https://min.io/docs/minio/container/index.html
## 容器化部署 单节点单磁盘模式，测试成功！
### MinIO Server
```sh
docker pull minio/minio

[root@c3 nginx]# docker images minio/minio
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
minio/minio   latest    e31e0721a96b   2 years ago   406MB

mkdir -p /opt/minio/{data,config}

docker run --name minio \
   -d --restart=always \
   -p 9000:9000 \
   -p 9001:9001 \
   -v /opt/minio/data:/data \
   -v /opt/minio/config:/root/.minio \
   -e "MINIO_ROOT_USER=root" \
   -e "MINIO_ROOT_PASSWORD=cfzx5488" \
   minio/minio server /data --console-address ":9001"

注：用户名至少3个字符，密码至少8个字符，否则run不成功！
```
浏览器访问：192.168.1.63:9001
建立bucket：cfzx（至少3个字符）
## Docker 部署 单节点多磁盘即纠删码模式，测试成功！
```sh
之前命令同上

mkdir -p /opt/minio/config

docker run --name minios \
   -d --restart=always \
   -p 9000:9000 \
   -p 9001:9001 \
   -v /opt/minio/data1:/data1 \
   -v /opt/minio/data2:/data2 \
   -v /opt/minio/data3:/data3 \
   -v /opt/minio/data4:/data4 \
   -v /opt/minio/data5:/data5 \
   -v /opt/minio/data6:/data6 \
   -v /opt/minio/data7:/data7 \
   -v /opt/minio/data8:/data8 \
   -e "MINIO_ROOT_USER=root" \
   -e "MINIO_ROOT_PASSWORD=cfzx5488" \
   minio/minio server /data{1...8} --console-address ":9001"
```
8个目录模拟8块硬盘，当然也可以4个目录模拟4块硬盘
```sh
[root@c3 nginx]# docker logs minios
```
![[Pasted image 20240530215010.png]]








