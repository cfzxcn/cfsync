---
date: 2024-04-09
---
# clickhouse特点
提供了sql结构化的查询语言;
clickhouse是一款分布式数据库;
clickhouse可以存储海量数据;
因为clickhouse是分布式存储海量数据,所以解决了高并发的问题;
clickhouse中的数据底层是列式存储
clickhouse 不仅可以管理自己的数据,也可以读取别人的数据,比如mysql , hdfs 网络和本地文件;
# 基础配置
## 关闭防火墙及selinux
## 配置/etc/security/limits.conf，以取消打开文件数限制
## 安装依赖
`yum install libtool *unixODBC*`

官网：https://clickhouse.com/
https://clickhouse.com/docs/en/getting-started/quick-start
https://clickhouse.com/docs/zh/getting-started/install
# ansible部署
ub20中：
root@ub20:~/playbook# cd roles;ansible-galaxy init clickhouse
- Role clickhouse was created successfully

root@ub20:~/playbook# ls roles/clickhouse/
README.md  defaults  files  handlers  meta  tasks  templates  tests  vars
# 下载安装包，以便离线安装
20.6.3及其后版本才支持explain；20.8出了一些新的引擎，可以实时同步mysql；
https://packages.clickhouse.com/rpm/stable/

## 安装包列表：
- `clickhouse-common-static` — ClickHouse编译的二进制文件。
- `clickhouse-server` — 创建`clickhouse-server`软连接，并安装默认配置服务
- `clickhouse-client` — 创建`clickhouse-client`客户端工具软连接，并安装客户端配置文件。
- `clickhouse-common-static-dbg` — 带有调试信息的ClickHouse二进制文件。

root@ub20:~/playbook# cd roles/clickhouse/files/
wget https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-dbg-22.9.7.34.x86_64.rpm

root@ub20:~/playbook# ll -h roles/clickhouse/files/
total 1.1G
drwxr-xr-x  2 root root  208 Apr 10 01:12 ./
drwxr-xr-x 10 root root  154 Apr  9 12:52 ../
-rw-r--r--  1 root root  87K Dec 16  2022 clickhouse-client-22.9.7.34.x86_64.rpm
-rw-r--r--  1 root root 237M Dec 16  2022 clickhouse-common-static-22.9.7.34.x86_64.rpm
-rw-r--r--  1 root root 813M Dec 16  2022 clickhouse-common-static-dbg-22.9.7.34.x86_64.rpm
-rw-r--r--  1 root root 112K Dec 16  2022 clickhouse-server-22.9.7.34.x86_64.rpm

# 离线安装
`rpm -ivh *.rpm`

安装clickhouse-server时，会提示：
enter password for ==default== user:
如不设密码就直接回车即可；如设了密码，每次客户端访问都要输入密码。

[root@localhost ~]# ll
total 1074544
-rw-r--r-- 1 root root     88071 Dec 16  2022 clickhouse-client-22.9.7.34.x86_64.rpm
-rw-r--r-- 1 root root 248224665 Dec 16  2022 clickhouse-common-static-22.9.7.34.x86_64.rpm
-rw-r--r-- 1 root root 851891397 Dec 16  2022 clickhouse-common-static-dbg-22.9.7.34.x86_64.rpm
-rw-r--r-- 1 root root    114011 Dec 16  2022 clickhouse-server-22.9.7.34.x86_64.rpm

[root@localhost ~]# yum localinstall *.rpm
或
[root@localhost ~]# rpm -ivh *.rpm
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:clickhouse-common-static-dbg-0:22################################# [ 25%]
   2:clickhouse-common-static-0:22.9.7################################# [ 50%]
   3:clickhouse-client-0:22.9.7.34-1  ################################# [ 75%]
   4:clickhouse-server-0:22.9.7.34-1  ################################# [100%]
ClickHouse binary is already located at /usr/bin/clickhouse
Symlink /usr/bin/clickhouse-server already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-server to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-client already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-client to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-local already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-local to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-benchmark already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-benchmark to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-copier already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-copier to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-obfuscator already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-obfuscator to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-git-import to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-compressor already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-compressor to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-format already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-format to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-extract-from-config already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-extract-from-config to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-keeper already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-keeper to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-keeper-converter to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-disks to /usr/bin/clickhouse.
Creating clickhouse group if it does not exist.
 groupadd -r clickhouse
Creating clickhouse user if it does not exist.
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse clickhouse
Will set ulimits for clickhouse user in /etc/security/limits.d/clickhouse.conf.
Creating config directory /etc/clickhouse-server/config.d that is used for tweaks of main server configuration.
Creating config directory /etc/clickhouse-server/users.d that is used for tweaks of users configuration.
Config file /etc/clickhouse-server/config.xml already exists, will keep it and extract path info from it.
/etc/clickhouse-server/config.xml has /var/lib/clickhouse/ as data path.
/etc/clickhouse-server/config.xml has /var/log/clickhouse-server/ as log path.
Users config file /etc/clickhouse-server/users.xml already exists, will keep it and extract users info from it.
Creating log directory /var/log/clickhouse-server/.
Creating data directory /var/lib/clickhouse/.
Creating pid directory /var/run/clickhouse-server.
 chown -R clickhouse:clickhouse '/var/log/clickhouse-server/'
 chown -R clickhouse:clickhouse '/var/run/clickhouse-server'
 chown  clickhouse:clickhouse '/var/lib/clickhouse/'
 groupadd -r clickhouse-bridge
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse-bridge clickhouse-bridge
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'
==Enter password for default user:== 
Password for default user is empty string. See /etc/clickhouse-server/users.xml and /etc/clickhouse-server/users.d to change it.
Setting capabilities for clickhouse binary. This is optional.
 chown -R clickhouse:clickhouse '/etc/clickhouse-server'

==ClickHouse has been successfully installed==.

Start clickhouse-server with:
 sudo clickhouse start

Start clickhouse-client with:
 clickhouse-client

Synchronizing state of clickhouse-server.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable clickhouse-server
Created symlink /etc/systemd/system/multi-user.target.wants/clickhouse-server.service -> /usr/lib/systemd/system/clickhouse-server.service.

## 确认安装：
[root@kylin ~]# rpm -qa | grep clickhouse
clickhouse-client-22.9.7.34-1.x86_64
clickhouse-server-22.9.7.34-1.x86_64
clickhouse-common-static-dbg-22.9.7.34-1.x86_64
clickhouse-common-static-22.9.7.34-1.x86_64

[root@kylin ~]# rpm -ql clickhouse-
clickhouse-client             clickhouse-common-static      clickhouse-common-static-dbg  clickhouse-server
[root@kylin ~]# rpm -ql clickhouse-common-static
/usr/bin/clickhouse
......

rpm或yum安装四个包后
配置文件在：/etc/clickhouse-client和/etc/clickhouse-server
库文件在：/var/lib/clickhouse
日志文件在：/var/log/clickhouse
命令文件在：/usr/bin
且会创建clickhouse用户
### 某些版本安装出现：error: unpacking of archive failed
[root@localhost tmp]# rpm -ivh clickhouse-common-static-21.9.6.24-2.x86_64.rpm --nodigest
warning: clickhouse-common-static-21.9.6.24-2.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID e0c56bd4: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:clickhouse-common-static-21.9.6.2################################# [100%]
error: unpacking of archive failed on file /usr/bin/clickhouse;66154444: cpio: read failed - No such file or directory
error: clickhouse-common-static-21.9.6.24-2.x86_64: install failed
**解决方法：换版本**
# 配置
## vim /etc/clickhouse-server/config.xml
`<!-- <listen_host>::</listen_host> -->` 默认只有本机可访问，即：localhost，
改为去掉注释，不对ip做限制，就可以其它主机访问此主机了：
`<listen_host>::</listen_host> `
单机安装只需要改这个就够了
# 启动clickhouse
systemctl start clickhouse-server
或：
clickhouse start|status|stop
## tail /var/log/clickhouse-server/clickhouse-server.err.log 报错

**\<Error> Application: DB::Exception: Listen [::]:8123 failed: Poco::Exception. Code: 1000, e.code() = 0, DNS error: EAI: Address family for hostname not supported (version 22.9.7.34 (official build))**
**解决方法：**
vim /etc/clickhouse-server/config.xml
<listen_host>::</listen_host>
改为：
<listen_host>0.0.0.0</listen_host>

**\<Error> Application: DB::Exception: Cannot lock file /var/lib/clickhouse/status. Another server instance in same directory is already running**
原因：之前clickhouse-server启动2次导致，所以，直接依据日志目录，查看这个status文件
**解决方法：**
[root@localhost ~]# cat /var/lib/clickhouse/status
PID: 59288
Started at: 2024-04-09 23:54:19
Revision: 54466

kill -9 59288
# 查看状态
[root@kylin ~]# clickhouse status
/var/run/clickhouse-server/clickhouse-server.pid file exists and contains pid = 59812.
The process with pid = 59812 is running.
# 连接clickhouse
clickhouse-client -m
-m：输入时可换行。分号代表语句结束
注：目前是以default用户在登录
# 数据库操作语句
show databases;
![[Pasted image 20240410024506.png|450]]
[root@kylin ~]# clickhouse-client --query 'show databases;'
INFORMATION_SCHEMA
default
information_schema
system
# 卸载包：
[root@localhost tmp]# rpm -e clickhouse-client 
[root@localhost tmp]# 
[root@localhost tmp]# rpm -e clickhouse-common-static-dbg 
[root@localhost tmp]# rpm -e clickhouse-server  
可以将包的配置文件，库等全部删除，删除很干净






