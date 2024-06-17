http://reboot.pro/files/file/303-tiny-pxe-server/
http://http.us.debian.org/debian/dists/Debian10.13/main/installer-amd64/current/images/netboot/
http://http.us.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/
### ipxe启动原理：  
网卡在进行pxe启动时会进行第一次dhcp请求，并附带client-arch选项。如果系统使用普通bios模式启动，那么client-arch就是0；如果是x86 uefi启动，就是7或者9；如果是arm64 uefi启动，就是11。  
dnsmasq会给为请求打上tag，例如`dhcp-match=set:x64-uefi,option:client-arch,7`，如果发现请求有这个tag，就推送对应的启动文件，例如`dhcp-boot=tag:aarch64-uefi,ipxe-x86_64.efi`。此时，主机就会使用tftp协议去下载ipxe，并且执行。  
当ipxe开始运行后，ipxe会再次进行dhcp请求，并且带上参数175。dnsmasq收到这个请求后，会推送boot.ipxe，就是下面这两行：  
`dhcp-match=set:ipxe,175` `dhcp-boot=tag:ipxe,boot.ipxe`  
而boot.ipxe就是ipxe的主配置文件。boot.ipxe里面还会引用其他配置文件，ipxe也将一一加载。  
至此，ipxe已经成功加载并且获取到了配置文件。
### 注：
iPXE默认不是support FTP server，待验证！
### 直接下载ipxe相关文件
wget -P /root/pxe/http/tftp http://boot.ipxe.org/ipxe.pxe
wget -P /root/pxe/http/tftp http://boot.ipxe.org/undionly.kpxe
wget -P /root/pxe/http/tftp http://boot.ipxe.org/ipxe.efi

或
### 下载并编译ipxe固件
[root@co7hw ~]# git clone https://github.com/ipxe/ipxe.git
Cloning into 'ipxe'...
remote: Enumerating objects: 60365, done.
remote: Counting objects: 100% (56/56), done.
remote: Compressing objects: 100% (37/37), done.
remote: Total 60365 (delta 28), reused 34 (delta 19), pack-reused 60309
Receiving objects: 100% (60365/60365), 18.22 MiB | 6.45 MiB/s, done.
Resolving deltas: 100% (45625/45625), done.

[root@localhost ~]# yum install xz-devel
[root@co7hw ~]# yum install gcc
\#  不装gcc下面无法编译

[root@localhost ~]# cd /root/ipxe/src

~~vim embed.ipxe~~
~~#!ipxe~~
~~dhcp~~
~~chain http://SERVER/menu.ipxe~~

[root@localhost ~]# make bin/undionly.kpxe ~~EMBED=embed.ipxe~~
[root@localhost ~]# make bin-x86_64-efi/ipxe.efi ~~EMBED=embed.ipxe~~
注：
![[Pasted image 20240331035636.png]]
在编译时，make bin/undionly.kpxe后如加EMBED=embed.ipxe，则启动时会有：http://SERVER/menu.ipxe... Error.....，所以编译时不要加：EMBED=embed.ipxe了

\#  执行完之后，在bin中会有一个undionly.kpxe文件，在bin-x86_64-efi会有ipxe.efi，之后会用到这两个文件
### 复制两个ipxe固件至/var/lib/tftpboot/ipxe/
[root@localhost ~]# cp /root/ipxe/src/bin/undionly.kpxe /var/lib/tftpboot/ipxe/
[root@localhost ~]# cp /root/ipxe/src/bin-x86_64-efi/ipxe.efi /var/lib/tftpboot/ipxe/
### /etc/dhcp/dhcpd.conf.ipxe
```
[root@co7hw ipxe]# cat /etc/dhcp/dhcpd.conf.ipxe 
option domain-name "sheying001.com";  # 必须用双引号，单引号不行,搜索的域后缀
option domain-name-servers 180.76.76.76,223.5.5.5; #dhcp服务器给主机指定的dns服务器地址，可写多个，逗号分隔
default-lease-time 21600;           #设置默认的IP租用期限,秒为单位
max-lease-time 43200;		# 设置最大的IP租用期限
log-facility local7;
#DHCPDARGS=ens33;  # 指定监听网卡
#DHCP configuration for PXE boot server
option client-architecture code 93 = unsigned integer 16;
subnet 192.168.1.0 netmask 255.255.255.0 {  #地址池，定义子网，有了这句，dhcpd服务就能正常启动了
	range 192.168.1.200   192.168.1.250; #可分配的起始IP-结束IP
	option routers 192.168.1.2; #网关
	next-server 192.168.1.79;  #添加tftp服务器地址,可以从这里下载启动文件
	if exists user-class and option user-class = "iPXE" {
		filename "ipxe/ipxe.cfg";
	} elsif option client-architecture = 00:00 {
		filename "ipxe/undionly.kpxe";
	} else {
		filename "ipxe/ipxe.efi";
	}
}
```
### 配置菜单
vim /var/lib/tftpboot/ipxe.cfg
### ub的ks文件
#### 官方说明
https://ubuntu.com/server/docs/install/autoinstall-reference
#### 已安装系统的应答文件位置
/var/log/installer/autoinstall-user-data

如果kernel...后不加initrd=initrd，启动client后报如下错误
![[Pasted image 20240405182042.png]]


### 启动过程截图
![[Pasted image 20240331045823.png]]
![[Pasted image 20240331055629.png]]
![[Pasted image 20240331060043.png]]
别人的：
![[Pasted image 20240405224527.png]]
### 报错：
![[Pasted image 20240331175149.png]]


ipxe.cfg中如用：linux  /kernel/debian125/debian-installer/amd64/linux
安装debian125时，报：
![[Pasted image 20240420070137.png]]
解决方法：linux改为kernel即可

![[Pasted image 20240420070702.png]]
The checksum of the file retrieved from <'""''http://192.168.1.79/ks/db125-preseed.cfg"''> fails to match the expected value of "". The file may be corrupt, or the provided checksums may be out of date.

![[Pasted image 20240420075601.png]]










