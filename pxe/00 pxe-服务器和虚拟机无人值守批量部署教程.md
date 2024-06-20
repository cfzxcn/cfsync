# 前言
作为运维工程师，给服务器装系统是家常便饭，如果偶尔装一两台，把U盘插到服务器上装就是了，如果是3-5台、十几台乃至上百台，还用U盘去装效率就太低下了，不仅浪费时间延误工期，而且一遍又一遍手工重复相同工作也单调乏味，容易出错。所以生产环境中会用 PXE+kickstart的方法，为多台服务器无人值守批量安装操作系统。

工作中，需要配置一台服务器作为服务端（我演示的是co7服务端），然后把要安装系统的多台客户端服务器与服务端服务器接到同一交换机中。客户端利用支持pxe协议的网卡向服务端请求分配ip地址，服务端使用dhcp服务给客户端分配ip地址，再用tftp服务将客户端需要的启动文件传输给客户端，客户端启动后按服务端各配置文件（kickstart应答文件）的配置安装操作系统，传输操作系统安装文件是由服务端的http/ftp/nfs服务实现

下面我用VM虚拟机演示下如何批量安装co7、Kylin-Server-V10和Ubuntu Server20.04

先看网络环境的配置：在虚拟网络编辑器中，可以使用NAT模式或仅主机模式（三种模式可理解为三种不同特点的路由器），它们的区别是：使用NAT模式的虚拟机可以访问互联网，使用仅主机模式的虚拟机不能访问互联网。这里以NAT模式为例，注意：要取消“使用本地的DHCP服务将ip地址分配给虚拟机”，因为我们要用一台配置好的服务端的dhcp服务来给客户端分配ip，而不是用vm的dhcp服务来分配ip

我这里服务端已经配置了NAT模式， 接下来要添加的客户端也要配置为NAT模式，这和实际工作环境类似，相当于接到了同一交换机上。

下面添加一个客户端。和普通安装不同有二，一是虚拟机设置中---选项---高级，固件类型改成uefi；
为什么要改成uefi呢？
因为真实物理服务器上硬盘容量都很大，一般是用几块或几十块硬盘组成raid磁盘阵列，往往都有几十T乃至几百T。所以分区格式肯定都是gpt，而gpt分区格式最好和uefi固件搭配。为了模拟真实工作环境，所以调整成uefi。
二是客户端启动时要使用网络启动；
# 教程特点
![[Pasted image 20240517205854.png]]
1. 适用性广
	1. 涵盖红帽系列7，8，9版本，如：CentOS/rhel7.9、Alma8.5、Rocky9.0；
	2. 国产系列如：AnolisOS7.9、openEuler-22.03、UOS-Server-20、Kylin-Server-V10
	3. Ubuntu Server系列：20.04、22.04；
	4. 基本覆盖了99%以上的国内linux服务器操作系统
	5. 注：如时间足够，会加上Debian系列：9、10、11、12版本
2. 面向实践：全部使用基于uefi固件的gpt分区；两种分区方式：传统分区、LVM；以下系统可在无/有外网环境下httpd/nfs安装：ub20，ub22，co7，Debian-9.13， AnolisOS-7.9，openEuler-22.03，UOS-Server-20-1060e（非1060a），Kylin-Server-V10-SP3，rhel7.9，alma85，Rocky8.5，Rocky9.0；
3. 少走弯路：实践中不用的技术不讲，如基于bios固件的操作系统部署、cobbler部署等。快速学会，早日应用在工作中
4. 拓展性：ks文件、user-meta详解及扩展。普通安装难以实现如：添加epel、nginx源、安装额外软件如docker、更改命令提示符颜色、添加alias别名等，但用应答文件可轻松实现，更高效
5. 教程、源码全有，购买本课程直接能用在工作中
6. 测试自动化：安装后的系统可实现xshell脚本自动化测试，进一步提升运维效率
# 适用人群
有一定linux基础，安装过linux操作系统，会分区，会yum安装软件，vim基本使用，简单shell基础
# PXE 和 Kickstart
首先两者是不同的技术
## PXE 预启动执行环境 （Pre-boot Execution Environment）
也被称为预执行环境，==作用：通过网络启动安装程序和操作系统==
是Intel公司研发的一种网络引导协议，基于Client/Server的网络模式（即客户端与服务器），只要是使用Inter的CPU，其BIOS都支持PXE技术。支持客户端通过网络从服务器下载启动程序及映像等，并由此通过网络启动操作系统。
### PXE 客户端
也就是要安装操作系统的服务器
要让PXE 客户端能够实现pxe引导，要求：网卡集成PXE固件，且主板支持网络引导。现在服务器基本都支持，只需在BIOS设置中允许从Network或LAN 启动即可。使用英特尔cpu的服务器在开机时，可按下F12键来强制从LAN启动，这将在需要时触发PXE引导过程
PXE 固件是网卡固件的一部分。提供了DHCP客户端和TFTP 客户端等功能模块。
在客户端启动过程中，BIOS 把 PXE 固件调入内存执行。然后由PXE 固件中的DHCP客户端模块向DHCP服务端请求分配IP地址，再用TFTP客户端模块\（trivial file transfer protocol\）或MTFTP\（multicast trivial file transfer protocol\）下载引导程序、最小内核等，然后在最小内核环境下通过HTTP协议或NFS协议或Ftp协议安装操作系统
PXE 客户端固件与BIOS 密切相关，可分为传统 PXE 客户端 和 EFI PXE 客户端。传统 PXE 客户端与传统 BIOS 一起工作，无法使用2T以上硬盘的空间，所以工作中不再使用；EFI PXE 客户端和EFI固件一起工作，可以使用2T以上硬盘的空间，所以现在是主流
而这两种模式使用的引导程序是不同的，可以在dhcp配置文件中来指定。另外如果希望客户端能够在 efi 模式下启动 iso 文件！ 要使用 GRUB 而不是 SYSLINUX。
![[Pasted image 20240518051608.png]]
![[Pasted image 20240518052042.png]]
### pxe 服务端
至少由三种服务组成：DHCP、TFTP、HTTP/FTP/NFS
- DHCP：负责给客户端分配ip地址、定位引导程序以及指定TFTP服务器的ip；
- TFTP（trivial file transfer protocol）：负责将PXE引导程序、内核文件vmlinuz、临时根文件镜像initrd等文件传输给客户机；
- HTTP/FTP/NFS：负责将安装操作系统所需的大文件传输给客户机
这三种服务可以同时搭建在同一台服务器上，也可以分别搭建在不同的服务器上；还可以分别部署在不同的平台上，如DHCP服务由Windows系统提供，TFTP搭建在Linux平台，HTTP由Unix系统提供服务；甚至分别可以由不同的设备提供，如路由器提供DHCP服务，手机提供TFTP服务，服务器提供HTTP服务
## UEFI PXE 引导顺序：
### 第一阶段：dhcp
client开机选择网络启动后（新购买的服务器或vm中新建的虚拟机不需要额外的设置直接启动即可；已安装操作系统的服务器启动时可调整 BIOS 中的 Boot 选项将 Network 或 LAN 设置为第一项，然后启动服务器），将网卡上的pxe client信息拷贝到内存中运行，发送dhcp广播报文，向server端的dhcpd服务发起获取IP地址的dhcp请求；server端分配ip，并告知客户端tftp服务端的ip地址。所以服务端要配置dhcp server服务
### 第二阶段：tftp
1. 网卡上存储空间有限，所以只能集成轻量级别的tftp客户端。客户端从第一阶段获取到ip后就会去tftp服务端获取引导文件bootloader（对于EFI BIOS，需要使用efi文件引导，为实现UEFI SecureBoot，大多数Linux使用shim.efi嵌套调用grubx64.efi来引导）。
2. 客户端获取并执行引导文件后，引导文件会加载同目录下的菜单配置文件(grub.cfg)并呈现GRUB 菜单
3. 在用户选择了grub 条目、或命中默认条目时（条目必须要包含内核以及initrd，还可包含其它一些引导选项，比如键盘、语言、远程repo、kickstart配置文件等等），client会加载tftp服务端提供的内核和initrd，所以服务器端还要搭建一个tftp服务端
### 第三阶段：ftp/http/nfs
client加载内核和initrd后，就要使用ftp/http/nfs中的一种来获取操作系统的安装文件、ks文件及各种参数，然后就可以实现自动化安装了
以上三个服务部署后，pxe服务器就搭建好了
注：如果在这个阶段没有提供ks文件，那么安装还是需要以交互方式手工去操作（半自动）。为了实现全自动安装，需要Kickstart来帮忙！
## pxe服务端准备
- 安装和配置 DHCP
- 安装和配置 TFTP
- 安装和配置 HTTPD/NGINX
- 准备引导文件
- 准备grub菜单
- 拷贝内核文件和虚拟根文件镜像initrd
- 挂载各发行版镜像文件；
- 准备ks文件
## Kickstart
在网络上启动安装程序是 PXE 的工作，而 Kickstart 的==作用是：自动化安装==
通过将两者结合起来，您可以按如下方式自动安装 ：
打开机器电源→ PXE 在网络上启动安装程序，→ Kickstart 自动进行安装

所谓Kickstart，就是为操作系统配置一个应答文件，然后安装程序就可以以应答文件方式运行以实现操作系统自动化安装。client向http服务端请求此文件，http服务端提供应答文件地址。红帽系发行版使用Kickstart 机制，早期的Ubuntu以及Debian使用preseed机制，ub由 autoinstall 所提供的 cloud-config 参数（即 user-data 文件）在安装过程中自动提供应答数据，从而令安装界面能够自动推进

KickStart 是一种无人值守安装方式，KickStart 的原理是通过记录安装过程中所需人工填写的各种参数，生成一个名为 \*.cfg 的文件；
在其后的安装过程中，当出现要求填写参数的情况时，安装程序会首先去查找 KickStart 生成的文件。当找到合适的参数时，就采用找到的参数，当没有找到合适的参数时，才需要手工干预。
**PXE+Kickstart** 结合起来使用时，可以实现大规模的自动化操作系统部署。具体来说，当一台计算机通过网络启动时，它会向网络上的PXE服务器发送请求，PXE服务器会回应并提供引导程序和Kickstart配置文件。计算机接收到这些信息后，会根据配置文件中的指示自动下载操作系统镜像并按照预设的选项进行安装，无需人工干预。
## PXE+Kickstart优点
- 规模化:能够同时装多台服务器
- 自动化:自动安装各项服务
- 远程实现:摆脱传统的U盘，光驱等介质
## 其它pxe
gPXE（Etherboot）和 iPXE（又称 gPXE 的后继版本）是 PXE 的实现之一，gPXE 的开发已经停止，取而代之的是 iPXE
Microsoft WDS（Windows Deployment Services）: WDS是Microsoft Windows操作系统中的一个功能，使用PXE来进行网络部署。它提供了一套完整的工具和服务，用于自动化部署和管理Windows操作系统。
Cobbler PXE是指使用Cobbler和PXE（Preboot Execution Environment，预启动执行环境）来实现自动化操作系统部署和管理的一种方法。不好用，不推荐
# 部署流程 
所有 VMs 均使用单一的网卡挂接为 NAT 方式，NAT 网络被用来模拟服务商网络。
一台 PXE Server 被运行在NAT网段中，使用co7.9 minimal 非图形化安装，提供 DHCP+TFTP+httpd 服务，这三者向 NewNode 提供无人值守安装的全部资源

## pxe客户端准备
在同一网段中建立若干新 VM 主机，并设置启动 BIOS 类型的 UEFI 方式而非传统方式。然后直接开机，令其自动查找 DHCP 获得 IP 地址，进入安装序列，完成全部安装任务后停留在启动就绪状态，从而达到了模拟的目的。







 





