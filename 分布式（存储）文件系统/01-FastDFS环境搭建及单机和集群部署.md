# 环境搭建
## OS: Kylin Linux Advanced Server V10
FastDSF需要用centos7及以上版本往上才能安装的
## 磁盘目录
数据存储位置：/dfsData
## 使用的软件
yum install gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel
- libfastcommon：FastDFS分离出的一些公用函数包
- FastDFS：FastDFS本体
- fastdfs-nginx-module：FastDFS和nginx的关联模块
- nginx
https://gh.api.99988866.xyz/https://github.com/happyfish100/libfastcommon/archive/refs/tags/V1.0.73.tar.gz
https://github.com/happyfish100/fastdfs-nginx-module/archive/refs/tags/V1.24.tar.gz
https://github.com/happyfish100/fastdfs/archive/refs/tags/V6.12.1.tar.gz
https://gh.api.99988866.xyz/https://github.com/happyfish100/libserverframe/archive/refs/tags/V1.2.3.tar.gz

https://gh.api.99988866.xyz/https://github.com/happyfish100/fastdfs-client-java/archive/refs/tags/V1.31.tar.gz
http://nginx.org/download/nginx-1.24.0.tar.gz
https://gh.api.99988866.xyz/https://github.com/happyfish100/fastDIR/archive/refs/tags/V5.3.0.tar.gz
```sh
root@ub22:~/playbook/roles/fastdfs/files# git clone https://github.com/happyfish100/fastdht.git
Cloning into 'fastdht'...
remote: Enumerating objects: 212, done.
remote: Counting objects: 100% (53/53), done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 212 (delta 23), reused 23 (delta 15), pack-reused 159
Receiving objects: 100% (212/212), 244.05 KiB | 428.00 KiB/s, done.
Resolving deltas: 100% (70/70), done.
#  去重的
```
### 安装libfastcommon
```bash
[root@es-node2 tmp]# cd libfastcommon-1.0.73/
[root@es-node2 libfastcommon-1.0.73]# ls
debian  doc  HISTORY  INSTALL  libfastcommon.spec  LICENSE  make.sh  php-fastcommon  README  src
[root@es-node2 libfastcommon-1.0.73]# ./make.sh && ./make.sh install
#  编译安装
```

```bash
......
gcc -Wall -Wformat-truncation=0 -Wformat-overflow=0 -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -g -O3 -c -fPIC -o sorted_array.lo sorted_array.c  
gcc -Wall -Wformat-truncation=0 -Wformat-overflow=0 -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -g -O3 -o libfastcommon.so -shared hash.lo chain.lo shared_func.lo ini_file_reader.lo logger.lo sockopt.lo base64.lo sched_thread.lo http_func.lo md5.lo pthread_func.lo local_ip_func.lo avl_tree.lo ioevent.lo ioevent_loop.lo fast_task_queue.lo fast_timer.lo locked_timer.lo process_ctrl.lo fast_mblock.lo connection_pool.lo fast_mpool.lo fast_allocator.lo fast_buffer.lo multi_skiplist.lo flat_skiplist.lo system_info.lo fast_blocked_queue.lo id_generator.lo char_converter.lo char_convert_loader.lo common_blocked_queue.lo multi_socket_client.lo skiplist_set.lo uniq_skiplist.lo json_parser.lo buffered_file_writer.lo server_id_func.lo fc_queue.lo sorted_queue.lo fc_memory.lo shared_buffer.lo thread_pool.lo array_allocator.lo sorted_array.lo -lm -ldl -lpthread
ar rcs libfastcommon.a hash.o chain.o shared_func.o ini_file_reader.o logger.o sockopt.o base64.o sched_thread.o http_func.o md5.o pthread_func.o local_ip_func.o avl_tree.o ioevent.o ioevent_loop.o fast_task_queue.o fast_timer.o locked_timer.o process_ctrl.o fast_mblock.o connection_pool.o fast_mpool.o fast_allocator.o fast_buffer.o multi_skiplist.o flat_skiplist.o system_info.o fast_blocked_queue.o id_generator.o char_converter.o char_convert_loader.o common_blocked_queue.o multi_socket_client.o skiplist_set.o uniq_skiplist.o json_parser.o buffered_file_writer.o server_id_func.o fc_queue.o sorted_queue.o fc_memory.o shared_buffer.o thread_pool.o array_allocator.o sorted_array.o
mkdir -p /usr/lib64
mkdir -p /usr/lib
mkdir -p /usr/include/fastcommon
install -m 755 libfastcommon.so /usr/lib64
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_define.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h locked_timer.h process_ctrl.h fast_mblock.h connection_pool.h fast_mpool.h fast_allocator.h fast_buffer.h skiplist.h multi_skiplist.h flat_skiplist.h skiplist_common.h system_info.h fast_blocked_queue.h php7_ext_wrapper.h id_generator.h char_converter.h char_convert_loader.h common_blocked_queue.h multi_socket_client.h skiplist_set.h uniq_skiplist.h fc_list.h locked_list.h json_parser.h buffered_file_writer.h server_id_func.h fc_queue.h sorted_queue.h fc_memory.h shared_buffer.h thread_pool.h fc_atomic.h array_allocator.h sorted_array.h /usr/include/fastcommon
```
### 安装FastDFS/tracker
[root@es-node2 tmp]# pwd
/tmp
[root@es-node2 tmp]# cd fastdfs-6.12.1/
[root@es-node2 fastdfs-6.12.1]# ls
client  conf             docker        HISTORY  init.d   make.sh     README.md     setup.sh  systemd  tracker
common  COPYING-3_0.txt  fastdfs.spec  images   INSTALL  php_client  README_zh.md  storage   test
[root@es-node2 fastdfs-6.12.1]# 
[root@es-node2 fastdfs-6.12.1]# ./make.sh && ./make.sh install
[root@es-node2 fastdfs-6.12.1]# cp -a /tmp/fastdfs-6.12.1/conf/http.conf /etc/fdfs/
[root@es-node2 fastdfs-6.12.1]# cp -a /tmp/fastdfs-6.12.1/conf/mime.types /etc/fdfs/
\#  拷贝以上两个文件是为了供nginx访问使用

**如果在 ./make.sh && ./make.sh install 时输出以下报错：**
gcc -Wall -Wformat-truncation=0 -Wformat-overflow=0 -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -g -O3 -c -o ../common/fdfs_global.o ../common/fdfs_global.c  -I../common -I/usr/local/include
In file included from ../common/fdfs_global.c:21:0:
../common/fdfs_global.h:17:10: 致命错误：sf/sf_global.h：没有那个文件或目录
 #include "sf/sf_global.h"
          ^~~~~~~~~~~~~~~~
编译中断。

**以上报错原因是：**
因为在安装 fastdfs-6.08 及以上版本时，从github 下载得安装包缺少了 libserverframe 网络框架
*PS：fastdfs-6.07 及之前得版本部署不需要单独编译安装 libserverframe，经测试 fastdfs-6.07 仅适配 libfastcommon-1.0.55 及以前得版本，如果安装 libfastcommon-1.0.56 及以上版本，编译 fastdfs-6.07 则会报错——（测试版本 libfastcommon：1.0.50-1.0.59；fastdfs：6.07、6.08）*---来自https://www.cnblogs.com/goujinyang/p/17823601.html
**解决方法：编译安装 libserverframe 网络框架**
[root@es-node2 tmp]# cd libserverframe-1.2.3/
[root@es-node2 libserverframe-1.2.3]# ls
debian  libserverframe.spec  LICENSE  make.sh  README.md  sample.conf  src
[root@es-node2 libserverframe-1.2.3]# ./make.sh && ./make.sh install
安装完成后返回重新编译 fastdfs，发现一切正常啦
### 安装fastdfs-nginx-module
作用是：通过http的方式访问storage server上的文件，当storage本机没有要找的文件时向源storage主机请求文件
解压后：
[root@es-node2 ~]# cp -a /tmp/fastdfs-nginx-module-1.24/src/mod_fastdfs.conf /etc/fdfs/
### 部署 nginx
wget http://nginx.org/download/nginx-1.24.0.tar.gz
```
    ./configure --prefix=/opt/nginx --conf-path=/opt/nginx/conf/nginx.conf --user=nginx --group=nginx --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/lock/nginx.lock --with-http_ssl_module --with-http_gzip_static_module --with-debug --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_random_index_module --with-http_secure_link_module --with-http_auth_request_module --with-threads --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_slice_module --with-file-aio --with-http_v2_module --add-module=/tmp/fastdfs-nginx-module-1.24/src/
```
# 单机部署
## /etc/hosts配置
192.168.1.112 dfs.cn，这个配置是为了内部集群识别使用
同时windows中，C:\Windows\System32\drivers\etc\hosts，也添加此条目，是为了浏览器使用
## tracker配置
[root@es-node2 ~]# egrep '^(port|base)' /etc/fdfs/tracker.conf
[root@es-node2 ~]# mkdir /dfsData
port = 22122
base_path = /dfsData     # 存储日志和数据的根目录，这是默认目录，改成：/dfsData
## storage配置
vim /etc/fdfs/storage.conf \# 需要修改的内容如下
port=23000   \#  storage服务端口（默认23000，一般不修改）
heart_beat_interval = 30  一般不改，心跳间隔
base_path=/dfsData    \# 数据和日志文件存储根目录，即storage基础目录
store_path0=/dfsData   \#  第一个虚拟磁盘存储目录，上传的文件在：/dfsData/data下；对应fastdfs的虚拟磁盘M00，store_path1对应磁盘M01
tracker_server=dfs.cn:22122   \#  tracker服务器IP和端口
http.server_port=80   \#  http访问文件的端口（默认8888，看情况修改，和nginx中保 持一致）
## 启动服务
```sh
#  永久关闭防火墙
systemctl disable firewalld.service
#  启动tracker
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
#  启动storage
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf
#  重启storage
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```
## 查看日志
[root@es-node2 ~]# ll -h /dfsData/logs/
总用量 40K
-rw-r--r-- 1 root root 6.6K  4月 17 14:19 storaged.log
-rw-r--r-- 1 root root  14K  4月 15 23:06 storaged.log.20240416
-rw-r--r-- 1 root root 3.7K  4月 16 21:19 storaged.log.20240417
-rw-r--r-- 1 root root 2.9K  4月 17 14:19 trackerd.log
-rw-r--r-- 1 root root 4.0K  4月 15 23:06 trackerd.log.20240416
-rw-r--r-- 1 root root 1.9K  4月 16 19:28 trackerd.log.20240417
## client测试
```sh
vim /etc/fdfs/client.conf
#  需要修改的内容如下
base_path=/dfsData
tracker_server=dfs.cn:22122   \#  tracker服务器IP和端口
#  保存后测试，返回ID表示成功，如：group1/M00/00/00/wKgDDWDtRu6AMPhBARB1pcz7xUs146.jpg

fdfs_upload_file /etc/fdfs/client.conf t2.jpg
#  上传文件，如报：Usage: fdfs_upload_file <config_file> <local_filename> [storage_ip:port] [store_path_index]，则：
[root@c3 nginx]# fdfs_upload_file /etc/fdfs/client.conf t2.jpg 
group1/M00/00/00/wKgBP2ZkzW-AfL34AACIpiHfjP8529.jpg
[root@c3 nginx]# ll /dfsData/data/00/00/wKgBP2ZkzW-AfL34AACIpiHfjP8529.jpg
-rw-r--r-- 1 root root 34982 Jun  9 05:30 /dfsData/data/00/00/wKgBP2ZkzW-AfL34AACIpiHfjP8529.jpg
#  查看上传的文件

#  删除文件
fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/wKgDCmD1LHaADXrMAAw3ED-01wQ106.jpg
```

```sh
[root@c3 nginx]# fdfs_test /etc/fdfs/client.conf upload t2.jpg 
This is FastDFS client test program v6.12.1

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.fastken.com/ 
for more detail.

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=192.168.1.63, port=23000

group_name=group1, ip_addr=192.168.1.63, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/wKgBP2ZkzeuAKxjWAACIpiHfjP8969.jpg
source ip address: 192.168.1.63
file timestamp=2024-06-09 05:32:27
file size=34982
file crc32=568298751
example file url: http://192.168.1.63/group1/M00/00/00/wKgBP2ZkzeuAKxjWAACIpiHfjP8969.jpg
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/wKgBP2ZkzeuAKxjWAACIpiHfjP8969_big.jpg
source ip address: 192.168.1.63
file timestamp=2024-06-09 05:32:27
file size=34982
file crc32=568298751
example file url: http://192.168.1.63/group1/M00/00/00/wKgBP2ZkzeuAKxjWAACIpiHfjP8969_big.jpg
```
上传成功之后生成文件全路径(配置nginx后可直接访问该路径，访问storage服务器下的图片)
## 配置nginx访问
```sh
vim /etc/fdfs/mod_fastdfs.conf
#  需要修改的内容如下：
tracker_server=dfs.cn:22122    #  tracker服务器IP和端口
url_have_group_name=true
store_path0=/dfsData
```
# 集群部署
在单机部署的基础上，添加如下内容，没说明的不用修改
## /etc/hosts中添加新主机信息
192.168.1.113 dfs.com
同时windows中，C:\Windows\System32\drivers\etc\hosts，也添加此条目
## storage\.conf/client.conf/mod_fastdfs.conf中追加
tracker_server=dfs.com:22122   \#  tracker服务器2的IP和端口
## 检测集群
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
\#  会显示会有几台服务器，有3台就会显示Storage1-Storage3的详细信息
## nginx 集群配置
/opt/nginx/conf/conf.d/fastdfs.conf  192.168.1.112的配置，即：dfs.cn
```yaml
upstream storage_server_group1 {
        server dfs.cn:80 weight=10;
        server dfs.com:80 weight=10;

}
server {
    listen 80;
    server_name localhost;

    location ~/group[0-9]/ {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://storage_server_group1;
    }

    location ~/group[0-9]/M00 {
        root /dfsData/data;
        ngx_fastdfs_module;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
#  以上两句待确定！
    }
}
```

/opt/nginx/conf/conf.d/fastdfs.conf  192.168.1.113的配置（单机这样配就行），即：dfs.com
```yaml
server {
    listen 80;  #  此端口与storage.conf中的http.server_port相同
    server_name localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        root  /opt/nginx/html;
#  以上三句待确定！
    }
}
```
两台主机按以上配置访问浏览器可看到上传的图片
http://dfs.cn/group1/M00/00/00/wKgBcGYdQrqAX92JAACIpiHfjP8642.jpg
或：
http://192.168.1.63:81/group1/M00/00/00/wKgBP2ZkzeuAKxjWAACIpiHfjP8969.jpg
[root@es-node2 ~]# ls /dfsData/data/00/00/wKgBcGYdQrqAX92JAACIpiHfjP8642.jpg 
/dfsData/data/00/00/wKgBcGYdQrqAX92JAACIpiHfjP8642.jpg











