# 挂载光盘
mkdir /var/www/html/iso/db11
mount /dev/sr1 /var/www/html/iso/db11
# 下载各版本netboot.tar.gz
## 方法一，直接下载：
==Debian 9 (Stretch)==
https://archive.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz
https://mirrors.aliyun.com/debian-archive/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz
https://cdimage.debian.org/cdimage/archive/9.13.0/amd64/iso-cd/debian-9.13.0-amd64-xfce-CD-1.iso
==Debian 10 (Buster)==
http://cdimage.debian.org/cdimage/archive/10.13.0/amd64/iso-cd/debian-10.13.0-amd64-xfce-CD-1.iso
http://cdimage.debian.org/cdimage/archive/10.13.0/amd64/iso-dvd/debian-10.13.0-amd64-DVD-1.iso
http://http.us.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/
http://ftp.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/netboot.tar.gz
http://ftp.nl.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/netboot.tar.gz
==Debian 11 (Bullseye)==
https://mirrors.ustc.edu.cn/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
https://mirrors.tuna.tsinghua.edu.cn/debian/dists/bullseye/main/installer-amd64/current/images/netboot/
*https://deb.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz*
http://http.us.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
http://ftp.nl.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
==Debian 12 (bookworm)==
http://http.us.debian.org/debian/dists/bookworm/main/installer-amd64/current/images/netboot
http://http.us.debian.org/debian/dists/Debian12.5/main/installer-amd64/current/images/netboot/
http://ftp.nl.debian.org/debian/dists/bookworm/main/installer-amd64/current/images/netboot/netboot.tar.gz
## 方法二，下载并解压
```sh
下载并解压image文件
export MIRROR=http://ftp.debian.org/debian/dists/bookworm
export ARCH=amd64
export IMAGE_URL=${MIRROR}/main/installer-$ARCH/current/images/netboot/netboot.tar.gz
现在下载 Debian 12 的启动镜像解压文件
wget -q -O - ${IMAGE_URL} | sudo tar xzf -
```
## 方法三
```sh
root@db11:~# apt-cache search debian-installer-11-netboot-amd64
查找
debian-installer-11-netboot-amd64 - Debian-installer network boot images for amd64
apt-get install debian-installer-11-netboot-amd64
安装

dpkg -s  debian-installer-11-netboot-amd64
已安装的包介绍，没安装的不行
dpkg -L debian-installer-11-netboot-amd64
查看包含的文件及目录
root@db11:~# du -hs /usr/lib/debian-installer
115M	/usr/lib/debian-installer
```
# 解压
[root@co7hw kernel]# mkdir debian11
[root@co7hw kernel]# cd debian11/
[root@co7hw debian11]# wget https://deb.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
[root@co7hw debian11]# ls
netboot.tar.gz
[root@co7hw debian11]# tar xf netboot.tar.gz 
[root@co7hw debian11]# ls
debian-installer  ldlinux.c32  netboot.tar.gz  pxelinux.0  pxelinux.cfg  version.info
\# 会在当前目录解压，而不是解压到一个新的目录
# 在/etc/dhcp/dhcpd.conf中添加：
filename "kernel/debian11/debian-installer/amd64/grubx64.efi";
systemctl restart httpd tftp dhcpd
# 将grub/grub.cfg做软链接
以便和grubx64.efi在同一目录，
[root@co7hw debian11]# cd debian-installer/amd64/
[root@co7hw amd64]# pwd
/var/lib/tftpboot/kernel/debian11/debian-installer/amd64
[root@co7hw amd64]# ln -sv grub/grub.cfg .
‘./grub.cfg’ -> ‘grub/grub.cfg’
[root@co7hw amd64]# ll
total 41628
-rw-r--r-- 1 root root   953048 Feb  5 19:50 bootnetx64.efi
drwxr-xr-x 2 root root     4096 Feb  5 19:50 boot-screens
drwxr-xr-x 3 root root       56 Apr  3 19:07 grub
lrwxrwxrwx 1 root root       13 Apr  3 19:21 ==grub.cfg -> grub/grub.cfg==
-rw-r--r-- 1 root root  3835328 Feb  5 19:50 ==grubx64.efi==
-rw-r--r-- 1 root root 30742977 Feb  5 19:50 initrd.gz
-rw-r--r-- 1 root root  7039552 Feb  5 19:50 linux
-rw-r--r-- 1 root root    42430 Feb  5 19:50 pxelinux.0
drwxr-xr-x 2 root root       21 Feb  5 19:50 pxelinux.cfg

# /var/lib/tftpboot/grub/grub.cfg配置
## interface=ens33
menuentry --hotkey=z 'z   Debian-10.13.0 ...菜单中
interface=ens33，不能写成：interface=eth0，否则装机后，网卡无法启动，需 /etc/network/interfaces中改为ens33才能启动网卡

mount /dev/sr1 /var/www/html/iso/db11，否则报：无效的镜像仓库
目前的问题：
![[Pasted image 20240419170524.png]]
# 预置文件preseed语法
## 安装后的debian的preseed文件
[root@debian ~]# cat /etc/debian_version 
9.13
[root@debian ~]# find / -iname '\*preseed\*'
/usr/share/osinfo/install-script/debian.org/debian-preseed-desktop.xml
/usr/share/osinfo/install-script/debian.org/debian-preseed-jeos.xml
/usr/share/osinfo/install-script/ubuntu.com/ubuntu-preseed-jeos.xml

```
debconf-set-selections -c /path/to/preseed.cfg
#  在使用前检查预置文件的语法
https://github.com/mbruzek/debian-preseed#check-the-syntax-of-preseed-files
```

```yml
### Partitioning
d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string regular
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman-auto/choose_recipe select atomic
# 分区结果如下，gpt，nolvm，自动分区
```
![[Pasted image 20240422020846.png]]
![[Pasted image 20240422020918.png]]
# 各种安装报错
## checksums
pxe/efidefault安装报错：
![[Pasted image 20240407231531.png]]
The checksum of the file retrieved from <..\*http:/192.168.1.79/ks/db11-preseed.cfg.""> fails to match theChecksum errorexpected value ofThe file may be corrupt, or the provided checksums may be out of date.
## package description for brltty-udeb
![[Pasted image 20240420144931.png]]
添加：apt-setup-udeb  apt-setup/services-select       multiselect
## Unknow language code
安装后进入安装界面的语言选择
tail -f /var/log/syslog 
![[Pasted image 20240420145336.png]]
d-i debian-installer/locale string zh_CN.UTF-8
d-i localechooser/supported-locales multiselect en_US.UTF-8, zh_CN.UTF-8
d-i keyboard-configuration/xkb-keymap select us
## release.gpg
添加：debian-installer/allow_unauthenticated=true preseed
## 请输入新用户的全名
![[Pasted image 20240420213306.png]]
d-i passwd/user-fullname cf
## 未找到内核模块
![[Pasted image 20240421234001.png]]
![[Pasted image 20240421234110.png]]
## 没有根文件系统/No matching physical volumes found
![[Pasted image 20240420203856.png]]
![[Pasted image 20240420212606.png]]
no matching physical volumes found
**Netboot安装包的initrd.gz无法识别本地安装的硬盘**
解决方式一：自己整合驱动到initrd.gz，推荐
1. 将网络启动和dvd本地启动的    initrd.gz   文件通过copio 解压；
2. 然后合并dvd启动的驱动“lib/modules/4.9.0-8-amd64/kernel/drivers/\*” 下的所有驱动模块合并；
3. 然后重新cpio打包即可！！
解决方式二：完全从官方mirror下来相应netboot版本的软件库到本地网络的软件仓库
解决方式三：直接使用公网的仓库安装。这种方式，对于新手，强烈推荐！
**解决方式一详细如下：**
```bash
[root@co7hw debian9]# pwd
/var/lib/tftpboot/kernel/debian9
[root@co7hw debian9]# 
[root@co7hw debian9]# wget https://archive.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz

[root@co7hw debian9]# ll -h
total 29M
-rw-r--r-- 1 root root 29M Jul 13  2020 netboot.tar.gz
[root@co7hw debian9]# tar xf netboot.tar.gz 
[root@co7hw debian9]# ls
debian-installer  ldlinux.c32  netboot.tar.gz  pxelinux.0  pxelinux.cfg  version.info
[root@co7hw debian9]# ls debian-installer/amd64/
bootnetx64.efi  boot-screens  grub  initrd.gz  linux  pxelinux.0  pxelinux.cfg
[root@co7hw debian9]# pwd
/var/lib/tftpboot/kernel/debian9

[root@co7hw debian9]# mkdir initrd
[root@co7hw debian9]# 
[root@co7hw debian9]# cd initrd/
[root@co7hw initrd]# 
[root@co7hw initrd]# cp /var/www/html/iso/db9/install.amd/initrd.gz .
[root@co7hw initrd]# ll -h
total 15M
-r--r--r-- 1 root root 15M Apr 22 00:42 initrd.gz
[root@co7hw initrd]# 
[root@co7hw initrd]# gunzip initrd.gz
[root@co7hw initrd]# ll -h
total 42M
-r--r--r-- 1 root root 42M Apr 22 00:42 initrd
[root@co7hw initrd]# 
[root@co7hw initrd]# mv initrd initrd-iso
[root@co7hw initrd]# ll -h
total 42M
-r--r--r-- 1 root root 42M Apr 22 00:42 initrd-iso

[root@co7hw initrd]# cp /var/lib/tftpboot/kernel/debian9/debian-installer/amd64/initrd.gz .
[root@co7hw initrd]# 
[root@co7hw initrd]# ll -h
total 66M
-rw-r--r-- 1 root root 24M Apr 22 00:44 initrd.gz
-r--r--r-- 1 root root 42M Apr 22 00:42 initrd-iso
[root@co7hw initrd]# 
[root@co7hw initrd]# gunzip initrd.gz
[root@co7hw initrd]# ll -h
total 111M
-rw-r--r-- 1 root root 69M Apr 22 00:44 initrd
-r--r--r-- 1 root root 42M Apr 22 00:42 initrd-iso

[root@co7hw initrd]# mv initrd initrd-net
[root@co7hw initrd]# ll -h
total 111M
-r--r--r-- 1 root root 42M Apr 22 00:42 initrd-iso
-rw-r--r-- 1 root root 69M Apr 22 00:44 initrd-net
[root@co7hw initrd]# 
[root@co7hw initrd]# 
[root@co7hw initrd]# mkdir -p {iso,net}
[root@co7hw initrd]# ll -h
total 111M
-r--r--r-- 1 root root 42M Apr 22 00:42 initrd-iso
-rw-r--r-- 1 root root 69M Apr 22 00:44 initrd-net
drwxr-xr-x 2 root root   6 Apr 22 00:45 iso
drwxr-xr-x 2 root root   6 Apr 22 00:45 net

[root@co7hw initrd]# cd iso
[root@co7hw iso]# cpio -i < ../initrd-iso
85814 blocks
[root@co7hw iso]# ll -h
total 20K
drwxr-xr-x  2 root root 4.0K Apr 22 00:45 bin
drwxr-xr-x  2 root root   33 Apr 22 00:45 dev
drwxr-xr-x 11 root root 4.0K Apr 22 00:45 etc
-rwxr-xr-x  1 root root  456 Apr 22 00:45 init
drwxr-xr-x  2 root root    6 Apr 22 00:45 initrd
drwxr-xr-x 13 root root 4.0K Apr 22 00:45 lib
drwxrwxr-x  2 root root   34 Apr 22 00:45 lib64
drwxr-xr-x  2 root root    6 Apr 22 00:45 media
drwxr-xr-x  2 root root    6 Apr 22 00:45 mnt
drwxr-xr-x  2 root root    6 Apr 22 00:45 proc
drwxr-xr-x  2 root root    6 Apr 22 00:45 run
drwxr-xr-x  2 root root 4.0K Apr 22 00:45 sbin
drwxr-xr-x  2 root root    6 Apr 22 00:45 sys
drwxrwxr-x  2 root root    6 Apr 22 00:45 tmp
drwxrwxr-x  6 root root   53 Apr 22 00:45 usr
drwxrwxr-x  6 root root   52 Apr 22 00:45 var

[root@co7hw iso]# cd ../net
[root@co7hw net]# ll -h
total 0
[root@co7hw net]# 
[root@co7hw net]# cpio -i < ../initrd-net
139483 blocks
[root@co7hw net]# ll -h
total 20K
drwxr-xr-x  2 root root 4.0K Apr 22 00:45 bin
drwxr-xr-x  2 root root   33 Apr 22 00:45 dev
drwxr-xr-x 12 root root 4.0K Apr 22 00:45 etc
-rwxr-xr-x  1 root root  456 Apr 22 00:45 init
drwxr-xr-x  2 root root    6 Apr 22 00:45 initrd
drwxr-xr-x 14 root root 4.0K Apr 22 00:45 lib
drwxrwxr-x  2 root root   34 Apr 22 00:45 lib64
drwxr-xr-x  2 root root    6 Apr 22 00:45 media
drwxr-xr-x  2 root root    6 Apr 22 00:45 mnt
drwxr-xr-x  2 root root    6 Apr 22 00:45 proc
drwxr-xr-x  2 root root    6 Apr 22 00:45 run
drwxr-xr-x  2 root root 4.0K Apr 22 00:45 sbin
drwxr-xr-x  2 root root    6 Apr 22 00:45 sys
drwxrwxr-x  2 root root    6 Apr 22 00:45 tmp
drwxrwxr-x  6 root root   53 Apr 22 00:45 usr
drwxrwxr-x  6 root root   52 Apr 22 00:45 var
[root@co7hw net]# cd ..
[root@co7hw initrd]# ls
initrd-iso  initrd-net  iso  net

[root@co7hw initrd]# cp -avr iso/lib/modules/4.9.0-13-amd64/kernel/drivers/* net/lib/modules/4.9.0-13-amd64/kernel/drivers/
cp: overwrite ‘net/lib/modules/4.9.0-13-amd64/kernel/drivers/acpi/fan.ko’? y
‘iso/lib/modules/4.9.0-13-amd64/kernel/drivers/acpi/fan.ko’ -> ‘net/lib/modules/4.9.0-13-amd64/kernel/drivers/acpi/fan.ko’
cp: overwrite ‘net/lib/modules/4.9.0-13-amd64/kernel/drivers/acpi/thermal.ko’? y
......
# 这里要复制很多文件
[root@co7hw initrd]# cd net
[root@co7hw net]# ls
bin  dev  etc  init  initrd  lib  lib64  media  mnt  proc  run  sbin  sys  tmp  usr  var
[root@co7hw net]# find | cpio -R 0:0 -o -H newc > ../initrd
164560 blocks

[root@co7hw net]# cd ..
[root@co7hw initrd]# ls
initrd  initrd-iso  initrd-net  iso  net
[root@co7hw initrd]# gzip initrd
[root@co7hw initrd]# ls
initrd.gz  initrd-iso  initrd-net  iso  net
[root@co7hw initrd]# pwd
/var/lib/tftpboot/kernel/debian9/initrd

[root@co7hw initrd]# ls /var/lib/tftpboot/kernel/debian9/debian-installer/amd64/
bootnetx64.efi  boot-screens  grub  initrd.gz  linux  pxelinux.0  pxelinux.cfg
[root@co7hw initrd]# mv /var/lib/tftpboot/kernel/debian9/debian-installer/amd64/initrd.gz{,.original}
[root@co7hw initrd]# ls /var/lib/tftpboot/kernel/debian9/debian-installer/amd64/
bootnetx64.efi  boot-screens  grub  initrd.gz.original  linux  pxelinux.0  pxelinux.cfg

[root@co7hw initrd]# cp initrd.gz /var/lib/tftpboot/kernel/debian9/debian-installer/amd64/
[root@co7hw initrd]# 
[root@co7hw initrd]# ls /var/lib/tftpboot/kernel/debian9/debian-installer/amd64/
bootnetx64.efi  boot-screens  grub  initrd.gz  initrd.gz.original  linux  pxelinux.0  pxelinux.cfg

[root@co7hw debian9]# pwd
/var/lib/tftpboot/kernel/debian9
[root@co7hw debian9]# du -hs initrd/
266M	initrd/
[root@co7hw debian9]# rm -rf initrd/
```
## 下图错误估计是preseed的问题
改为db125-preseed.cfg就行了
![[Pasted image 20240422025624.png]]
![[Pasted image 20240422025738.png]]
## debian 10
```sh
[root@co7hw initrd]# cp -a iso/lib/modules/4.19.0-21-amd64/kernel/drivers/* net/lib/modules/4.19.0-21-amd64/kernel/drivers/
cp: overwrite ‘net/lib/modules/4.19.0-21-amd64/kernel/drivers/acpi/fan.ko’? y
cp: overwrite ‘net/lib/modules/4.19.0-21-amd64/kernel/drivers/acpi/thermal.ko’? y

[root@co7hw keyrings]# pwd
/var/lib/tftpboot/kernel/debian10/initrd/net/usr/share/keyrings
wget http://http.us.debian.org/debian/dists/buster/InRelease
wget http://http.us.debian.org/debian/dists/buster/Release
wget http://http.us.debian.org/debian/dists/buster/Release.gpg
```
## debian 11
```sh
[root@co7hw initrd]# cp iso/lib/modules/5.10.0-18-amd64/kernel/drivers/* net/lib/modules/4.9.0-13-amd64/kernel/drivers/
```
## debian 12
```bash
[root@co7hw initrd]# cp -avr iso/lib/modules/6.1.0-18-amd64/kernel/drivers/* net/lib/modules/6.1.0-18-amd64/kernel/drivers/
cp: overwrite ‘net/lib/modules/6.1.0-18-amd64/kernel/drivers/acpi/battery.ko’? y
‘iso/lib/modules/6.1.0-18-amd64/kernel/drivers/acpi/battery.ko’ -> ‘net/lib/modules/6.1.0-18-amd64/kernel/drivers/acpi/battery.ko’
cp: overwrite ‘net/lib/modules/6.1.0-18-amd64/kernel/drivers/acpi/fan.ko’? y
```

![[Pasted image 20240422041229.png]]
![[Pasted image 20240422081858.png]]

# db10  grub.cfg配置
linux行加入：net.ifnames=0 biosdevname=0 interface=ens33，则安装系统后网卡无ip，ip a 查看为ens33，但/etc/network/interfaces中为eth0，而非ens33，去除以上指令即可安装后成功获取ip

# db问题
![[Pasted image 20240510193622.png]]







