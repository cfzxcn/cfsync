## dhcp简介
DHCP：动态主机配置协议（Dynamic Host Configuration Protocol），应用在局域网中。作用：为网络中的主机自动分配ip地址、gateway、DNS等信息。是udp协议。
dhcp基于c/s模式：客户端/服务端模式，服务端用udp协议的67端口（在服务端看到67端口，就说明开启了dhcp服务），客户端用udp协议的68端口
**常见可提供dhcp服务的设备：**
- dhcp服务器（win server、linux服务器）
- 家用宽带路由器
- vmware软件
## dhcp工作流程
使用DHCP时，在网络上首先必须有一台DHCP服务器，而其他的计算机则是DHCP客户端，当DHCP客户端程序发出一个信息，要求一个动态IP地址时，DHCP服务器将根据目前配置的IP地址池，从中提供一个可供使用的IP地址和子网掩码给客户端，以下图就是DHCP的工作流程！
client向server申请ip地址，需要4个数据报文的往返：

上图是DHCP==发现阶段==，即DHCP客户端寻找DHCP服务器的阶段，DHCP客户端以==广播方式==发送DHCP Discover包，因为客户端不知道服务器的IP地址，在网络上每台主机都会收到此广播包，但是只有DHCP服务器才能响应

上图是DHCP的提供阶段，即DHCP服务器提供IP地址的阶段，收到DHCP客户端发送的DHCP DiscoVer的DHCP服务器，都会做出响应。这些DHCP服务器从尚未出租的IP地址中挑选一个给客户端，向客户端发送一个包含IP地址和其他设置的DHCP Offer包

上图是DHCP的选择阶段，DHCP客户端选择某台DHCP服务器提供IP地址阶段，当客户端收到多台DHCP发送的DHCP offer包，DHCP客户端只接受其中一台DHCP服务器的数据，然后以==广播方式==回应DHCP服务器DHCP Request，通知自己选择的DHCP服务器，当局域网中所有的DHCP服务器收到客户端发送的DHCP Request信息，通过查看包，确定是否是选择了自己IP，如果选择的是自己，则会发送一个确认包．否则，不进行响应

上图是DHCP的确认阶段，也就是DHCP服务器确认所提供的IP地址信息阶段，同时另外的没有被选择的DHCP服务器都将回收曾经提供的IP地址
## co7 安装和配置dhcpd服务端 
co7名为dhcp，co8名为dhcp-server
在vm中关闭dhcp服务，以避免和我们自己配置的dhcp服务冲突
```sh
[root@pxe ~]# rpm -qa dhcp
[root@pxe ~]# rpm -qi dhcp
未安装软件包 dhcp 
[root@pxe ~]# yum install dhcp -y
[root@pxe ~]# rpm -qa dhcp
dhcp-4.2.5-83.el7.centos.1.x86_64
[root@pxe ~]# rpm -ql dhcp
/etc/NetworkManager
/etc/NetworkManager/dispatcher.d
/etc/NetworkManager/dispatcher.d/12-dhcpd
/etc/dhcp/dhcpd.conf
/etc/dhcp/dhcpd6.conf
/etc/dhcp/scripts
/etc/dhcp/scripts/README.scripts
/etc/openldap/schema/dhcp.schema
/etc/sysconfig/dhcpd
/usr/bin/omshell
/usr/lib/systemd/system/dhcpd.service
/usr/lib/systemd/system/dhcpd6.service
/usr/lib/systemd/system/dhcrelay.service
/usr/sbin/dhcpd
/usr/sbin/dhcrelay
/usr/share/doc/dhcp-4.2.5
/usr/share/doc/dhcp-4.2.5/dhcpd.conf.example   #  模板文件
/usr/share/doc/dhcp-4.2.5/dhcpd6.conf.example
......
/var/lib/dhcpd   # 数据库
/var/lib/dhcpd/dhcpd.leases   #  租约文件
/var/lib/dhcpd/dhcpd6.leases
[root@pxe ~]# tail /var/log/messages    # 查看日志
[root@pxe ~]# systemctl start dhcpd
#  现在启动会报错，因为配置文件中还未配置subnet
```
### dhcpd主配文件：/etc/dhcp/dhcpd.conf 配置
- man 5 dhcpd.conf
- 主配文件可以直接手工编辑或用模板文件生成：
`cp/usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf`
- 该配置文件的关键字不区别大小写；指令以分号结尾
- subnet开头的是子网配置，不在花括号中的指令是全局配置，subnet中的指令和全局配置中的指令冲突时，以subnet为准，因为subnet的优先级更高
- option：指定client的选项；不以option开头的控制dhcp服务器自身的行为
#### 全局配置：对所有网段都起作用
##### option domain-name "baidu.com";
全局选项，该选项可有可无，指定搜索的DNS域名后缀，可以是ip或域名。必须用双引号，单引号不行，定义后，cat /etc/resolv.conf就会出现：search baidu.com，以后ping www，就相当于ping www.baidu.com
##### option domain-name-servers 180.76.76.76, 223.5.5.5;
server给client指定的dns服务器地址，可以是ip或域名。可写多个，逗号分隔，180.76.76.76 是百度dns地址。该配置项同样将体现在客户机的/etc /resolv.conf配置文件中（如“nameserver 202.106.0.20”）
##### default-lease-time 21600;
默认租约时间。指客户机通过DHCP获取IP后，这个IP使用多久。如果client没有设置期望租期，就按默认租期。秒为单位，默认600秒，因为租期一半时间即每5分钟就要申请续约一次，可增大此值，因为这个时间太短会增加网络流量。21600是6小时，43200是12小时，86400是1天，172800是2天，3天(3\*24\*60\*60=259200秒），518400是6天，691200是8天。设置长租期，客户端获得的ip较固定。
##### max-lease-time 43200;
最大租约时间，正常情况下，如果客户机在default-lease-time快到期时会向DHCP续租。如果在default-lease-time期间，客户机关机/死机了，default-lease-time时间到了，DHCP服务器并不会立即回收这个IP，将继续等待，等待的时间就是max-lease-time。如果在max-lease-time到期前还不来续租，那就回收IP以便其它client使用。max-lease-time应大于default-lease-time，一般可设置为默认租约时间的2倍
##### log-facility local7;		日志级别
##### authoritative;
本地权威dhcp服务器。如果该dhcp服务器是本地官方dhcp就将此选项打开，避免其它dhcp服务器的干扰。如果有多台dhcp服务器，客户端要ip，只能从这台dhcp服务器要，即：本地所有主机获得的ip都是由本dhcp发放。使用此选项后，vm中-编辑-虚拟网络编辑器-勾选“使用本地dhcp服务将ip地址分配给虚拟机”也不影响使用
##### next-server 192.168.1.3; 
下一个要访问的地址，就是指定tftp服务器地址，可以从这里下载启动文件，也可以在subnet中指定子网的tftp服务器地址
##### filename "BOOTX64.EFI";
告知客户端bootloader启动引导文件的名称（指定要下载的tftp上的PXE引导程序文件），是全局配置，但也可用于subnet中
#### subnet 设置子网属性
```yml
subnet 192.168.1.0 netmask 255.255.255.0 {
#  声明要分配的网段地址（发放ip地址的范围），定义子网（作用域），有了这句，dhcpd服务就能正常启动了
range 192.168.1.100   192.168.1.200; 
#  地址池，准备为客户端分配的自定义IP地址范围，注意：如果dhcpd服务启动失败，可能是因为主配文件中没有配置本地网段的subnet，主配文件中可以有多个网段的subnet，但其中一个必须是本地网段的subnet，即该subnet地址池的ip和ip a中的网卡ip是同一网段
option routers 192.168.1.2; # 分配给客户端的网关地址
option broadcast-address  # 为客户端设定广播地址，可选，注释掉后用默认的
ntp-server            # 为客户端设定网络时间服务器IP地址，co7测试不生效
```
子网也可以设置租约时长，如果和全局设置一样就没必要写了 
#### 地址排除
==要求排除192.168.1.220-192.168.1.230之间的ip==
```yml
subnet 192.168.1.0 netmask 255.255.255.0 {
	range 192.168.1.200   192.168.1.219; 
	range 192.168.1.231   192.168.1.253;
	option routers 192.168.1.2;   # 网关
}
```
#### 保留地址
**在server端，配置client端mac地址和ip地址的静态绑定**
在IP租约到期后，如果无法续订租约，client只能重新获得一个新IP使用，但是公司有些设备的IP地址是不希望变化的，比如公司文件服务器、打印服务器等。那么在这种环境中既想使用DHCP管理公司IP，又想实现部分机器的IP永久不变，那么怎么实现呢。DHCP的作者在写DHCP的时候也想到了这个问题，提出了保留IP的概念，就是将某些IP保留，然后设备来获取IP时，根据其MAC地址做匹配，将对应的IP分给它即可
```sh
host cf-131-client {
  hardware ethernet 00:50:56:28:be:0d;  #  指定网卡接口类型和MAC地址
  filename  "vmunix.cf-131-client";   # 
  fixed-address 192.168.1.231;   #  分配给客户端一个固定的地址
  server-name “bbs.st.com”;  # 发给客户端下一个服务器地址，即找完dhcp后，客户端再去找谁
# option host-name “cf.st.com”; # 为客户端指定计算机名称，此功能只能使用在保留地址中，要求client的计算机名配置文件中将对应字段删除，即删除客户端/etc/hostname中的hostname，删除后client可能要重启
}

#  host 是保留ip地址的关键字，用于设置单个主机的网络属性，通常用于为网络打印机或个别服务器分配固定的IP地址（保留地址），这些主机的共同特点是：每次动态获取的IP地址必须相同，以确保服务的稳定性；host声明可以独立使用，也可以放在某个subnet声明中。cf-131-client 是自定义名
#  00:50:56:28:be:0d  是原192.168.1.131服务器ens33的MAC地址
#  192.168.1.231：这个ip，一定来自192.168.1.0网段，但不受range 192.168.1.100   192.168.1.200的限制，一般自己指定ip，这个ip不在地址池里，因为地址池里的ip有可能被dhcpd分配给其它服务器
#  租期、网关、dns都能单独指定

systemctl restart dhcpd
```
## server启动dhcpd并和client测试
```bash
[root@pxe ~]# systemctl enable --now dhcpd
# 虽然是yum安装的dhcp，但并未自动设置为开机自启
[root@pxe ~]# netstat -tnlpu | grep dhcpd
udp        0      0 0.0.0.0:67              0.0.0.0:*                           1263/dhcpd
```
### dhcp客户端配置
安装co7系统后**默认**就安装了dhcp客户端 ![[Pasted image 20240426145012.png|350]]
只要手工配置ip，dhclient默认不开启，所以手工配置ip时，此程序进程不存在
[root@co7hw ~]# ps aux | grep [d]hclient
[root@co7hw ~]# 
#### 方法一：dhcp客户端静态ip改为动态
只要网卡配置文件中使用了BOOTPROTO="dhcp"，重启网卡后，dhclient就开启了，就自动申请新的ip了
```sh
[root@co7hw ~]# netstat -tnlpu | grep dhclient
[root@co7hw ~]#
#  未使用BOOTPROTO="dhcp"前
[root@co7hw ~]# netstat -tnlpu | grep dhclient
udp        0      0 0.0.0.0:68              0.0.0.0:*                           70515/dhclient
# 使用BOOTPROTO="dhcp"， ifdown ens33;ifup ens33 后 或 nmcli conn reload;nmcli conn up ens33 后
```
dhclient使用固定的68端口，不同于传统的客户端，传统的服务的客户端都是随机端口，只有server端端口是固定的
![[Pasted image 20240426145643.png|450]]
在server端可看到：[root@pxe ~]# cat /var/lib/dhcpd/dhcpd.leases
```sh
[root@pxe ~]# cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.2.5
lease 192.168.1.41 {   #  给client分配的ip
  starts 5 2024/04/26 07:41:17;  # 这个时间是utc时间，+8小时，就是我们的时间
  ends 5 2024/04/26 13:41:17;   #  结束时间减开始时间是6小时，就是dhcpd.conf中配置的default-lease-time 21600
  cltt 5 2024/04/26 07:41:17;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:b6:97:21;  # 这个是客户端的mac地址
  client-hostname "co7hw";   #  client的主机名
}
server-duid "\000\001\000\001-\276\002*\000PV5\312\371";
```
#### 方法二：手工运行dhclient触发dhcp客户端获取ip地址
[root@co7hw network-scripts]# vim ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
**BOOTPROTO=dhcp**
**BOOTPROTO=static**     #  静态和动态ip地址共存
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
NAME=ens33
UUID=50950712-c814-4618-9311-d9d412e5d2f6
DEVICE=ens33
ONBOOT=yes
IPADDR='192.168.1.79'
PREFIX=24
GATEWAY=192.168.1.2
DNS1=223.5.5.5
DNS3=192.168.1.2
IPV6_PRIVACY=no
```sh
[root@co7hw network-scripts]# dhclient -d ens33
#  dhclient [-d] [ens33]：以前台方式运行，可以查看获得ip的过程；单独执行dhclient命令则默认以后台方式运行，无法看到此过程
Internet Systems Consortium DHCP Client 4.2.5
Copyright 2004-2013 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/ens33/00:0c:29:b6:97:03
Sending on   LPF/ens33/00:0c:29:b6:97:03
Sending on   Socket/fallback
DHCPDISCOVER on ens33 to 255.255.255.255 port 67 interval 5 (xid=0x5e80e14d)
DHCPREQUEST on ens33 to 255.255.255.255 port 67 (xid=0x5e80e14d)
DHCPOFFER from 192.168.1.3
DHCPACK from 192.168.1.3 (xid=0x5e80e14d)
bound to 192.168.1.42 -- renewal in 10493 seconds.   #  获得ip

[root@co7hw ~]# cat /var/lib/dhclient/dhclient.leases
#  client中查看，生成一个获取dhcp ip地址的文件(数据库)
lease {
  interface "ens33";
  fixed-address 192.168.1.42;
  option subnet-mask 255.255.255.0;
  option routers 192.168.1.2;
  option dhcp-lease-time 21600;
  option dhcp-message-type 5;
  option domain-name-servers 180.76.76.76,223.5.5.5;
  option dhcp-server-identifier 192.168.1.3;
  option domain-name "st.cn";
  renew 5 2024/04/26 10:55:02;
  rebind 5 2024/04/26 13:15:07;
  expire 5 2024/04/26 14:00:07;
}

[root@co7hw ~]# ip r
default via 192.168.1.2 dev ens33 proto static metric 100 
default via 192.168.1.2 dev ens38 proto dhcp metric 101
#  可看到，dhcp获取的网关和static获取的网关，static获取的网关metric值较小，优先级较高

[root@co7hw ~]# cat /etc/resolv.conf
# Generated by NetworkManager
search baidu.com
nameserver 223.5.5.5
nameserver 192.168.1.2
nameserver 180.76.76.76
#  也能看到从dhcp获得的domain-name、dns信息
```
##### 其它命令
###### dhclient -r ens33
-r：释放ip；释放当前租约并停止正在运行的DHCP客户端。dhclient -r ens33 执行后，ifconfig ens33所有ip就没有了（包括静态ip），要想再获取静态ip，可：ifdown ens33;ifup ens33，要想再获取动态ip，则再：dhclient，当然前提是网卡配置文件中BOOTPROTO=dhcp
###### killall dhclient;dhclient ens33
dhclient重新获取ens33的dhcp信息
## 允许DHCP/tftp/ftp服务通过Linux防火墙
**proxy-dhcp**端口是传播TFTP服务器IP地址所必需的
firewall-cmd --permanent --add-service={dhcp,proxy-dhcp,tftp,ftp}
firewall-cmd –reload
## 排错
### expecting numeric value
tailf /var/log/messages 报：
```sh
Apr 26 13:59:07 pxe dhcpd: /etc/dhcp/dhcpd.conf line 19: expecting numeric value.
Apr 26 13:59:07 pxe dhcpd: #011range 192.168.1.40 
Apr 26 13:59:07 pxe systemd: dhcpd.service: main process exited, code=exited, status=1/FAILURE
Apr 26 13:59:07 pxe dhcpd:                            ^
Apr 26 13:59:07 pxe systemd: Failed to start DHCPv4 Server Daemon
```
原因：格式错误，我是从ob中复制到配置文件的
```sh
[root@pxe ~]# cat /etc/dhcp/dhcpd.conf -A
......
subnet 192.168.1.0 netmask 255.255.255.0 {$
^Irange 192.168.1.40 M-BM- 192.168.1.49;$
^Irange 192.168.1.80M-BM-  192.168.1.89;$
#  option broadcast-address 192.168.1.255;$
^Ioption routers 192.168.1.2;$
}$
......
# 可看到，多了几处win的格式，^I  和  M-BM-，删除它们
```
systemctl restart dhcpd即可。



