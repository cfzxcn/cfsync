https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-2F0D4696-3681-4C13-9552-5280C6406376.html
https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-F594C1E5-E385-4F32-A88E-4BFF81B8AFBD.html
*大多数 UEFI 固件本身包含 PXE 支持，允许从 TFTP 服务器引导。固件可直接加载用于 UEFI 系统的 ESXi 引导加载程序 mboot.efi，而不需要 PXELINUX 等其他软件。*
注：查询官方文档，如果 ESXi 主机仅运行旧版 BIOS 固件，那么可配置pxe的菜单文件：PXELINUX，所以可从菜单安装esxi；如果 ESXi 主机运行 UEFI 固件，那么应该无法配置pxe的菜单文件，只能直接在dhcpd.conf中指定filename "kernel/esxi7/mboot.efi" 进行安装
## 一、bootx64.efi 和crypto64.efi
将 efi/boot/bootx64.efi 文件从 ESXi 安装程序 ISO 映像复制到 TFTP 服务器的 /tftpboot/kernel/esxi7 文件夹中，并且将文件重命名为 mboot.efi。对于 7.0 Update 3 及更高版本，还请将 efi/boot/crypto64.efi 文件复制到 /tftpboot 文件夹
```
mount /dev/sr4 /mnt/
# vm虚拟机中光盘挂载：VMware-VMvisor-Installer-7.0U3g-20328353.x86_64.iso，然后mount到/mnt

mkdir /var/lib/tftpboot/kernel/esxi{7,8}
cp /mnt/efi/boot/bootx64.efi /var/lib/tftpboot/kernel/esxi7/mboot.efi
cp /mnt/efi/boot/crypto64.efi /var/lib/tftpboot/kernel/esxi7/

注：新版本的 mboot.efi 通常可以引导旧版本的 ESXi，但旧版本的 mboot.efi 可能无法引导新版本的 ESXi。如果您计划配置不同的主机以引导不同版本的 ESXi 安装程序，请使用最新版本中的 mboot.efi。
```
## 二、配置 DHCP 服务器
https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-91E32FD0-A33C-4302-9FAB-B52B8A5CEFBC.html
```
        filename "kernel/esxi7/mboot.efi";
        filename "kernel/esxi8/mboot.efi";

systemctl restart httpd tftp dhcpd
```
## 三、拷贝ESXi 映像内容到新创建的目录esxi7.0U3g
```
mkdir /var/lib/tftpboot/kernel/esxi7.0U3g
mkdir /var/lib/tftpboot/kernel/esxi8.0U1
cp -a /mnt/* /var/lib/tftpboot/kernel/esxi7.0U3g
cp -a /var/www/html/iso/esxi8.0U1/* /var/lib/tftpboot/kernel/esxi8.0U1

注：不能用挂载esxi镜像，然后在/var/lib/tftpboot/kernel/esxi8/boot.cfg中用prefix=http://192.168.1.3/iso/esxi8.0U1/的方式，否则安装过程中会报：Fatal error: 24 (TFTP error)，因为是用tftp来传输的
```
## 四、修改/var/lib/tftpboot/kernel/esxi7.0U3g/boot.cfg文件
#### a. 修改prefix=为：
```
prefix=kernel/esxi7.0U3g
prefix=kernel/esxi8.0U1
#  其中， kernel/esxi7.0U3g是安装程序文件相对于 TFTP 服务器 root 目录（/var/lib/tftpboot/）的路径名称
```
#### b. 如果 kernel= 和 modules= 行中的文件名以正斜杠 (/) 字符开头，请删除该字符
```
:%s#/##g
#  vim中执行如上
```
#### c. 如果 kernelopt= 行包含字符串 cdromBoot，请只移除该字符串
#### d. （可选）指定安装脚本位置
**（可选）** 对于脚本式安装，在 boot.cfg 文件中内核命令后的一行添加 `kernelopt` 选项以指定安装脚本的位置：kernelopt=ks=http://192.168.1.79/ks/esxi7.cfg
#### 修改后的文件如下：
```
bootstate=0
title=Loading ESXi installer
timeout=5
prefix=kernel/esxi7.0U3g
kernel=b.b00
kernelopt=ks=http://192.168.1.79/ks/esxi7.cfg
#kernelopt=runweasel
modules=jumpstrt.gz --- useropts.gz --- features.gz --- k.b00 --- uc_intel.b00 --- uc_amd.b00 --- uc_hygon.b00 --- procfs.b00 --- vmx.v00 --- vim.v00 --- tpm
.v00 --- sb.v00 --- s.v00 --- atlantic.v00 --- bnxtnet.v00 --- bnxtroce.v00 --- brcmfcoe.v00 --- elxiscsi.v00 --- elxnet.v00 --- i40en.v00 --- iavmd.v00 --- 
icen.v00 --- igbn.v00 --- ionic_en.v00 --- irdman.v00 --- iser.v00 --- ixgben.v00 --- lpfc.v00 --- lpnic.v00 --- lsi_mr3.v00 --- lsi_msgp.v00 --- lsi_msgp.v0
1 --- lsi_msgp.v02 --- mtip32xx.v00 --- ne1000.v00 --- nenic.v00 --- nfnic.v00 --- nhpsa.v00 --- nmlx4_co.v00 --- nmlx4_en.v00 --- nmlx4_rd.v00 --- nmlx5_co.
v00 --- nmlx5_rd.v00 --- ntg3.v00 --- nvme_pci.v00 --- nvmerdma.v00 --- nvmetcp.v00 --- nvmxnet3.v00 --- nvmxnet3.v01 --- pvscsi.v00 --- qcnic.v00 --- qedent
v.v00 --- qedrntv.v00 --- qfle3.v00 --- qfle3f.v00 --- qfle3i.v00 --- qflge.v00 --- rste.v00 --- sfvmk.v00 --- smartpqi.v00 --- vmkata.v00 --- vmkfcoe.v00 --
- vmkusb.v00 --- vmw_ahci.v00 --- bmcal.v00 --- crx.v00 --- elx_esx_.v00 --- btldr.v00 --- esx_dvfi.v00 --- esx_ui.v00 --- esxupdt.v00 --- tpmesxup.v00 --- w
easelin.v00 --- esxio_co.v00 --- loadesx.v00 --- lsuv2_hp.v00 --- lsuv2_in.v00 --- lsuv2_ls.v00 --- lsuv2_nv.v00 --- lsuv2_oe.v00 --- lsuv2_oe.v01 --- lsuv2_
oe.v02 --- lsuv2_sm.v00 --- native_m.v00 --- qlnative.v00 --- trx.v00 --- vdfs.v00 --- vmware_e.v00 --- vsan.v00 --- vsanheal.v00 --- vsanmgmt.v00 --- tools.
t00 --- xorg.v00 --- gc.v00 --- imgdb.tgz --- basemisc.tgz --- resvibs.tgz --- imgpayld.tgz
build=7.0.3-0.55.20328353
updated=0

说明：
title=STRING    将引导加载程序标题设置为 STRING
prefix=STRING  （可选）在尚未以 / 或 http:// 开头的 kernel= 和 modules= 命令中，在每个 FILEPATH 前面添加 STRING
kernel=FILEPATH   将内核路径设置为 FILEPATH
kernelopt=STRING    将 STRING 附加到内核引导选项
modules=FILEPATH1 --- FILEPATH2... --- FILEPATHn    列出要加载的模块，用三个连字符 (`---`) 分隔
```
## 五、/var/www/html/ks/esxi7.cfg（ks文件从官方复制）
安装或升级脚本可驻留在以下位置之一：
- FTP 服务器
- HTTP/HTTPS 服务器
- NFS 服务器
- USB 闪存驱动器
- CD-ROM 驱动器
https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-341A83E4-2A6C-4FB9-BE30-F1E19D12947F.html
https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-61A14EBB-5CF3-43EE-87EF-DB8EC6D83698.html
https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-C3F32E0F-297B-4B75-8B3E-C28BD08680C8.html
以下文字从以上链接复制：
ESXi 安装程序包含一个默认安装脚本，该脚本可对第一个检测到的磁盘执行标准安装。
可以使用 `ks=file://etc/vmware/weasel/ks.cfg`引导选项指定默认 ks.cfg 文件的位置

使用 ks.cfg 脚本安装 ESXi 时，默认根密码为 `myp@ssw0rd`。
**修改后的ks文件可实现：自动设置系统密码、自动添加许可证、自动配置静态ip**
```
#
# Sample scripted installation file
#

# Accept the VMware End User License Agreement
vmaccepteula

# Set the root password for the DCUI and Tech Support Mode
rootpw CFzx5488
#  密码必须至少7位，否则安装报错

# Install on the first local disk available on machine
install --firstdisk --overwritevmfs

# Set the network to DHCP on the first network adapter
# network --bootproto=dhcp --device=vmnic0
#  这条是默认的，使用dhcp分配ip

# Set the network on the first network adapter
network --bootproto=static --device=vmnic0 --ip=192.168.8.71 --netmask=255.255.255.0 --vlanid=0 --gateway=192.168.8.1 --hostname=esxi7 --nameserver=192.168.8.1
#  新添加的

#vmserialnum
vmserialnum --esx=HN2X0-ODH5M-M78Q1-780HH-CN214
# 添加许可证

# reboot after install
reboot

# enable & start remote ESXi Shell (SSH)
#vim-cmd hostsvc/enable_ssh
#vim-cmd hostsvc/start_ssh
#  安装提示：vim-cmd，未知命令，所以注释掉
# enable & start ESXi Shell (TSM)
#vim-cmd hostsvc/enable_esx_shell
# vim-cmd hostsvc/start_esx_shell

# enable High Performance
# esxcli system settings advanced set --option=/Power/CpuPolicy --string-value="High Performance"
#  安装提示：esxcli，未知命令，所以注释掉

# /bin/chkconfig ntpd on
#  安装提示：/bin/chkconfig，未知命令，所以注释掉

# A sample post-install script
%post --interpreter=python --ignorefailure=true
import time
stampFile = open('/finished.stamp', mode='w')
stampFile.write( time.asctime() )
```
![[Pasted image 20240330230513.png]]
以上为不支持的命令，之前测试得出的结果，是安装过程中的弹窗
## 六、指定是否希望所有 UEFI 主机引导同一安装程序
#### a. 同一安装程序
复制或链接boot.cfg到/var/lib/tftpboot/kernel/esxi7目录
```
cp /var/lib/tftpboot/kernel/esxi7.0U3g/boot.cfg /var/lib/tftpboot/kernel/esxi7
```
如果不做上步，client启动时提示：
在pxe安装esxi8时测试，先复制：cp /var/lib/tftpboot/kernel/esxi7.0U3g/boot.cfg /var/lib/tftpboot/kernel/esxi7，然后直接修改/var/lib/tftpboot/kernel/esxi7/boot.cfg也是可以的
#### b. 不同安装程序
1. 创建 /tftpboot 的子目录，并以目标主机的 MAC 地址 (01-mac_address_of_target_ESXi_host) 命名，例如 01-23-45-67-89-0a-bc。
2. 将主机 boot.cfg 文件的副本（或链接）置于此目录中，例如 /tftpboot/01-23-45-67-89-0a-bc/boot.cfg。
## 七、配置虚拟机pxe安装esxi7/8

![[Pasted image 20240330200901.png]]  ![[Pasted image 20240524171714.png]]  ![[Pasted image 20240524171906.png]]

ESXi7-pxe
![[Pasted image 20240330200958.png]]   ![[Pasted image 20240330201027.png]]
![[Pasted image 20240525105450.png]]
注：pxe安装时，由于dhcp.conf中配置的是192.168.1.0网段，即NAT，所以pxe客户端的网络连接也应该是NAT模式，但我现有pxe集群用的是192.168.8.0网段即桥接模式，所以安装后要改为桥接
新建虚拟机后，在虚拟机设置中---选项---高级---默认就是uefi，
![[Pasted image 20240330210314.png]]
译：*ESXi 7.0.3 ui11在评估节点运行60天。评估期结束后要使用ESxi 7.0.3，必须注册vMuare产品license。要管理您的服务器，请从web浏览器导航到服务器的域名或IP地址，或使用直接控制用户界面。重新启动前请拆卸安装介质。重启服务器以启动ESXi 7.0.3。后来在ks文件中可以自动注册许可证了*

按回车重启后不会再次安装，成功了！




