https://www.openeuler.org/zh/download/get-os/
下载的是Standard ISO，安装时是图形化的，但软件选择中没有图形化；Everything ISO没下载测试

装完后anaconda-ks.cfg显示：
\# Use graphical install
graphical
但init 5无法切换到图形化，很奇怪

https://mirror.sjtu.edu.cn/openeuler/
上海交大下载链接
vm虚拟机安装时选择centos8
![[Pasted image 20240329010430.png]]
root   CFzx5488；cf   CFzx5488

## 登录后输出
Authorized users only. All activities may be monitored and reported.
root@192.168.1.226's password: 

Authorized users only. All activities may be monitored and reported.
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Mar 29 01:23:42 2024


Welcome to 5.10.0-182.0.0.95.oe2203sp3.x86_64

System information as of time: 	Fri Mar 29 01:24:12 CST 2024

System load: 	0.00
Processes: 	168
Memory used: 	7.7%
Swap used: 	0%
Usage On: 	4%
IP address: 	192.168.1.226
Users online: 	2

[root@localhost ~]# cat /etc/os-release 
NAME="openEuler"
VERSION="22.03 (LTS-SP3)"
ID="openEuler"
VERSION_ID="22.03"
PRETTY_NAME="openEuler 22.03 (LTS-SP3)"
ANSI_COLOR="0;31"

[root@localhost ~]# uname -r
5.10.0-182.0.0.95.oe2203sp3.x86_64
[root@localhost ~]# hostname
localhost.localdomain
安装时没有默认没有主机名，我也没输入，但装完有hostname
[root@localhost ~]# echo $LANG
C.UTF-8
[root@localhost ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.3Gi       495Mi       2.7Gi       9.0Mi       321Mi       2.8Gi
Swap:          2.0Gi          0B       2.0Gi

[root@localhost ~]# systemctl get-default
multi-user.target

[root@localhost ~]# netstat -tnlpu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2678/sshd: /usr/sbi 
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      838/rpcbind         
tcp6       0      0 :::22                   :::*                    LISTEN      2678/sshd: /usr/sbi 
tcp6       0      0 :::111                  :::*                    LISTEN      838/rpcbind         
udp        0      0 0.0.0.0:57138           0.0.0.0:*                           838/rpcbind         
udp        0      0 0.0.0.0:68              0.0.0.0:*                           1355/dhclient       
udp        0      0 0.0.0.0:111             0.0.0.0:*                           838/rpcbind         
udp        0      0 127.0.0.1:323           0.0.0.0:*                           887/chronyd         
udp6       0      0 :::60262                :::*                                838/rpcbind         
udp6       0      0 :::111                  :::*                                838/rpcbind         
udp6       0      0 ::1:323                 :::*                                887/chronyd