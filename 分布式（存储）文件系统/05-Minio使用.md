# minio的web控制台管理
## 创建访问Access key（相当于用户名）和Secret key（相当于密码）
用于程序访问，它们是访问数据的凭据
web控制台左侧菜单第二项Access keys---右侧：Create access key---直接点击：Create
- Access key：83aUf8BfTDMGI2AyM4EK
- Secret key：EYBMQ2sVpaAwSCtqLsy793RnTHeqqGNHGnh80cfe
注意：此处的Secret key是一次性显示，需要立即复制保存，否则只能重新生成新的Access key
# minio客户端MC的使用
## mc介绍
https://min.io/docs/minio/linux/reference/minio-mc.html
https://min.io/docs/minio/linux/reference/minio-mc-admin.html
MC是MinlO客户端命令行工具MinIO Client（mc)。类似mysqld的客户端mysql；可部署在没有安装minio的单独主机上
MC可用于访问MinlO的文件，使用方法类似于常用的UNIX命令：如：ls,cat,cp,rm,diff,find等
它支持文件系统和兼容Amazon S3的云存储服务（AWS Signature v2和v4)
## 部署mc
### 包安装MinIO Client
```sh
dnf install https://dl.min.io/client/mc/release/linux-amd64/mcli-20240524090849.0.0-1.x86_64.rpm
mcli alias set myminio/ http://MINIO-SERVER MYUSER MYPASSWORD
```
### 二进制部署mc
```sh
wget https://dl.minio.org.cn/client/mc/release/linux-amd64/mc
#  国内镜像下载
wget https://dl.minio.io/client/mc/release/linux-amd64/mc
#  官网下载
install mc /usr/local/bin/mc
chmod +x /usr/local/bin/mc
mc alias set myminio/ http://MINIO-SERVER MYUSER MYPASSWORD
```
### shell脚本部署二进制mc
![[Pasted image 20240418232902.png|800]]
![[Pasted image 20240531092051.png|347]]
![[Pasted image 20240531092143.png|425]]
![[Pasted image 20240531092217.png|83]]
![[Pasted image 20240418233034.png|675]]
### docker部署mc
https://min.io/download#/docker
![[Pasted image 20240418232638.png]]
## mc命令
```sh
mc -h

USAGE:                                                                                                                                                                                       
  mc [FLAGS] COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

mc COMMAND -h
#  mc的COMMAND帮助，如：mc alias -h
```
### mc常用命令
```yaml
ls # 列出文件和文件夹
mb # 创建一个存储桶或一个文件夹
cat # 显示文件和对象内容
pipe # 一个STDIN重定向到一个对象或者文件或者STDOUT
share # 生成用于共享的URL
cp # 拷贝文件和对象
mirror # 给存储桶和文件夹做镜像
find # 基于参数查找文件
diff # 对两个文件夹或者存储桶比较差异
rm # 删除文件和对象
events # 管理对象通知
watch # 监视文件和对象的事件
policy # 管理访问策略
config # 管理mc配置文件
update # 检查软件更新
version, -v # 输出版本信息
```
### 开启mc命令自动补全功能
```sh
mc --autocompletion
#  需要重新登录才能生效，exit
```
### 查看默认连接信息
```sh
[root@es-node4 ~]# mc alias ls
gcs  
  URL       : https://storage.googleapis.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v2
  Path      : dns

local
  URL       : http://localhost:9000
  AccessKey : 
  SecretKey : 
  API       : 
  Path      : auto

play 
  URL       : https://play.min.io
  AccessKey : Q3AM3UQ867SPQQA43P2F
  SecretKey : zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
  API       : S3v4
  Path      : auto

s3   
  URL       : https://s3.amazonaws.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v4
  Path      : dns
```
### 配置本地local连接（还未测试）
```sh
mc alias set local http://192.168.1.3:9090 minio minioadmin
#  配置本地local连接
mc alias ls        或        mc config host ls
#  查看配置是否成功
[root@es-node4 ~]# mc admin info local/ 
mc: <ERROR> Unable to get service status. Get "http://localhost:9000/minio/admin/v3/info?metrics=false": dial tcp [::1]:9000: connect: connection refused.

[root@es-node4 ~]# mc config host add cfminio http://192.168.1.220:9090 minio minioadmin
#  #连接minio
Added `cfminio` successfully.
[root@es-node4 ~]# mc alias ls
cfminio
  URL       : http://192.168.1.220:9090
  AccessKey : minio
  SecretKey : minioadmin
  API       : s3v4
  Path      : auto
......
```

![[Pasted image 20240418233518.png]]
![[Pasted image 20240531094056.png]]
![[Pasted image 20240531094130.png]]
![[Pasted image 20240531094158.png]]
![[Pasted image 20240531094404.png]]
![[Pasted image 20240531094431.png]]
![[Pasted image 20240531094512.png]]

![[Pasted image 20240531094230.png]]

![[Pasted image 20240418233801.png|925]]
![[Pasted image 20240531100212.png|950]]
![[Pasted image 20240531100244.png]]
![[Pasted image 20240531100305.png]]
![[Pasted image 20240531100323.png]]
![[Pasted image 20240531100345.png]]

![[Pasted image 20240531101259.png]]
![[Pasted image 20240531101232.png]]

![[Pasted image 20240531100601.png]]
![[Pasted image 20240531101618.png]]
![[Pasted image 20240531101813.png]]
![[Pasted image 20240531101923.png]]
![[Pasted image 20240531102143.png]]
![[Pasted image 20240531102205.png]]
![[Pasted image 20240531102227.png]]

![[Pasted image 20240531100625.png]]
![[Pasted image 20240531102439.png]]
### [MinIO 支持诊断工具](https://www.minio.org.cn/docs/minio/linux/operations/checklists/hardware.html#id19)











