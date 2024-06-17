# 安装和配置 TFTP 服务
## TFTP介绍
Trivial File Transfer Protocol（简单文件传输协议，是文件传输协议FTP的简化版本），是一个基于UDP协议实现的用于在客户机和服务器之间进行简单文件传输的协议，适合于小文件传输的场合。TFTP服务默认由xinetd服务进行管理，使用UDP 端口69。
**TFTP特点**：
1. TFTP是一种开放协议，缺乏安全性，没有加密机制，与TFTP服务端通信时不需要身份认证
2. TFTP使用UDP（性能好，没有三次握手，可靠性低）作为传输层协议，TFTP使用69端口
3. TFTP只有5个指令可以执行（rrg，wrq，data，ack，error）
4. 基于c/s模式，co7中，tftp-server：服务端包； tftp：客户端包
5. 容量小，tftp客户端只有50多K，所以可以集成在pxe网卡芯片内
## co7/co8 安装和使用 TFTP服务端
```sh
yum install vim bash-completion net-tools rsync tree lrzsz
systemctl disable --now postfix firewalld
hostnamectl set-hostname pxe
setenforce 0;sed -i.bak '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config
# 初始化

[root@localhost ~]# yum list *tftp*
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
可安装的软件包
syslinux-tftpboot.noarch       4.05-15.el7                                 base
tftp.x86_64                            5.2-22.el7                                   base
tftp-server.x86_64                 5.2-22.el7                                   base

[root@localhost ~]# yum install -y tftp-server
[root@localhost ~]# rpm -qa | grep tftp
tftp-server-5.2-22.el7.x86_64

[root@localhost ~]# rpm -ql tftp
未安装软件包 tftp 
[root@localhost ~]# 
[root@localhost ~]# rpm -ql tftp-server 
/etc/xinetd.d/tftp
/usr/lib/systemd/system/tftp.service
/usr/lib/systemd/system/tftp.socket
/usr/sbin/in.tftpd
/usr/share/doc/tftp-server-5.2
/usr/share/doc/tftp-server-5.2/CHANGES
/usr/share/doc/tftp-server-5.2/README
/usr/share/doc/tftp-server-5.2/README.security
/usr/share/man/man8/in.tftpd.8.gz
/usr/share/man/man8/tftpd.8.gz
/var/lib/tftpboot  # 数据目录
[root@localhost ~]# systemctl start tftp
[root@localhost ~]# systemctl status tftp
# 输出可看出：status tftp=status tftp.service
[root@localhost ~]# systemctl status tftp.service
● tftp.service - Tftp Server
   Loaded: loaded (/usr/lib/systemd/system/tftp.service; indirect; vendor preset: disabled)
   Active: active (running) since 四 2024-04-25 16:01:39 CST; 3s ago
     Docs: man:in.tftpd
 Main PID: 1724 (in.tftpd)
   CGroup: /system.slice/tftp.service
           └─1724 /usr/sbin/in.tftpd -s /var/lib/tftpboot

4月 25 16:01:39 localhost.localdomain systemd[1]: Started Tftp Server.
[root@localhost ~]# netstat -tnlup | grep -w 69
udp6       0      0 :::69                   :::*                                1/systemd
# 装完 tftp-server并启动后，只有这一个进程，systemd代替tftp-server监听69端口
[root@localhost ~]# systemctl is-enabled tftp
indirect
[root@localhost ~]# systemctl enable --now  tftp
# 可能不需要执行此条，因为yum安装后的软件一般都会自动设置为开机自启
[root@localhost ~]# systemctl is-enabled tftp
indirect
[root@localhost ~]# systemctl is-enabled tftp.socket 
enabled
# 所以，操作 tftp.socket或 tftp.service都行。为了简单，直接操作tftp吧！如果为了更清楚的操作，操作tftp.socket。因为 tftp.service最终是调用了 tftp.socket
[root@localhost ~]# reboot
# 重启后发现 systemctl status tftp.socket 是 active (listening)状态，而systemctl status tftp.service则是inactive (dead)状态，但这是正常的，69端口启用可证明是正常的，只要有人访问tftp-server，就会自动激活tftp.service

[root@pxe ~]# yum install xinetd
# 目的是重启xinetd使更改后的tftp设置生效，只重启tftp.socket或tftp.service无效，所以以后和tftp-server一起装吧！
[root@pxe ~]# netstat -tnlpu | grep 69
udp         0      0 0.0.0.0:69          0.0.0.0:*                        1379/xinetd         
udp6       0      0 :::69                   :::*                                1/systemd
#  安装xinetd后多了一个xinetd 进程
```
## TFTP服务端特殊之处
1. 直接使用tftp.socket 而非tftp.service，无人访问时，tftp.service是dead状态
2. 用systemd代替 tftp-server监听69端口，有人访问时，systemd会唤醒tftp.service，tftp.service是active(running)状态
3. 用xinetd重启tftp-server来使用配置生效
## co7 安装和使用 TFTP客户端
pxe网卡都内置了tftp客户端，为了测试tftp服务端能否正常工作，所以在另一台虚拟机中安装tftp客户端来进行测试，这是pxe工作流程的非必要项
```bash
[root@co7hw ~]# yum install -y tftp
[root@co7hw ~]# tftp 192.168.1.3
tftp> get 3.cfg
[root@co7hw ~]# ll 3.cfg 
-rw-r--r-- 1 root root 0 Apr 25 16:53 3.cfg
#  此时发现下载的文件大小为0，说明有问题，解决方法：server端关闭firewalld，或：firewall-cmd --permanent --add-service=tftp

[root@co7hw ~]# tftp 192.168.1.3
tftp> get 3.cfg
Error code 0: Permission denied
解决方法：server端：chmod -R 757 /var/lib/tftpboot/，默认该目录权限为755

[root@co7hw ~]# tftp 192.168.1.3
tftp> put install.sh 
Error code 1: File not found
解决方法：server端/etc/xinetd.d/tftp中改为：
server_args          = -s /var/lib/tftpboot -c
disable                 = no
或：sed -i '14s#yes#no#'/etc/xinetd.d/tftp
且安装了xinetd并systemctl restart xinetd生效，重启tftp.socket或tftp.service无效
```
## tftp服务端配置文件/etc/xinetd.d/tftp说明
```conf
	protocol		= udp   #  TFTP默认使用UDP协议
	wait			= yes   #  no表示客户机可以多台一起连接，yes表示客户机只能一台一台连接
	user			= root   
	server			= /usr/sbin/in.tftpd
	server_args		= -s /var/lib/tftpboot -c    #  指定TFTP根目录(引导文件的存储路径)
	disable			= no   #  no表示开启TFTP服务
```
 

tftp日志在哪里




