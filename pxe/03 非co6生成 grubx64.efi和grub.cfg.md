---
date: 2024-04-19
---
# 生成UEFI 引导文件：grubx64.efi
## 方法一：从openEuler-22.03-LTS-SP3拷贝
```bash
[root@pxe ~]# mount /dev/sr0 /var/www/html/iso/openEuler-22.03-LTS-SP3/
[root@pxe ~]# ll /var/www/html/iso/openEuler-22.03-LTS-SP3/EFI/BOOT/
总用量 4323
-r-xr-xr-x 1 root root  953104 12月 28 14:37 BOOTX64.EFI
dr-xr-xr-x 2 root root    2048 1月   2 15:43 fonts
-r--r--r-- 1 root root    1463 1月   2 15:43 grub.cfg
-r-xr-xr-x 1 root root 2605024 12月 28 14:20 grubx64.efi
-r-xr-xr-x 1 root root  862376 12月 28 14:37 mmx64.efi
-r--r--r-- 1 root root    1104 1月   2 15:46 TRANS.TBL
[root@pxe ~]# cp /var/www/html/iso/openEuler-22.03-LTS-SP3/EFI/BOOT/grubx64.efi /var/lib/tftpboot/grub/
#  拷贝后注意此文件是否有读权限，如没有则：chmod +r /var/lib/tftpboot/grub/grubx64.efi
#  此grubx64.efi引导后，菜单顶部显示：version：2.06
```
### 注：其它发行版经测试均不可用
mount /var/www/html/iso/ub20/ubuntu-20.04-live-server-amd64.iso /media/
cp /media/EFI/BOOT/grubx64.efi /var/lib/tftpboot/grub
用ub20的ISO的grubx64.efi启动会报错如下：
![[Pasted image 20240427015206.png]]
拷贝后grubx64.efi的权限默认为：444，更改权限为777依然报错，大小为：1419128
ub22的grubx64.efi报错同ub20，大小为：1480584

/var/www/html/iso/co7/EFI/BOOT/grubx64.efi，大小为：1097544
/var/www/html/iso/uos20-1060e/EFI/BOOT/grubx64.efi，大小为：2533272
/var/www/html/iso/kylin-Server-V10-SP3/EFI/BOOT/grubx64.efi，大小为：2502656
使用以上三个发行版的grubx64.efi，启动后会缺失菜单顶部，显示不完整
## 方法二：从ub20/22生成
>ub20/22方法一样，经测试都行
```sh
root@ub22:~# apt-get download grub-efi-amd64-signed
#  在ub22中下载 UEFI 引导文件：grub-efi-amd64-signed
root@ub22:~# ls
grub-efi-amd64-signed_1.187.6+2.06-2ubuntu14.4_amd64.deb 
root@ub22:~# mkdir grub
root@ub22:~# dpkg -x grub-efi-amd64-signed_1.187.6+2.06-2ubuntu14.4_amd64.deb grub
#  解压 grub 安装包到 grub 文件夹
root@ub22:~# ll grub/usr/lib/grub/x86_64-efi-signed/
total 7008
drwxr-xr-x 2 root root     101 Oct  2  2023 ./
drwxr-xr-x 3 root root      31 Oct  2  2023 ../
-rw-r--r-- 1 root root 2275208 Oct  2  2023 gcdx64.efi.signed
-rw-r--r-- 1 root root 2291592 Oct  2  2023 grubnetx64.efi.signed
-rw-r--r-- 1 root root 2598792 Oct  2  2023 grubx64.efi.signed
-rw-r--r-- 1 root root      17 Oct  2  2023 version

root@ub22:~# scp grub/usr/lib/grub/x86_64-efi-signed/grubnetx64.efi.signed 192.168.1.3:/var/lib/tftpboot/grub/grubx64.efi
#  拷贝后注意此文件是否有读权限，如没有则：chmod +r /var/lib/tftpboot/grub/grubx64.ef

# 以下生成的bootx64.efi不是必须的，但经测试也能正确引导
root@ub22:~# apt-get download shim.signed
root@ub22:~# ls
grub  grub-efi-amd64-signed_1.187.6+2.06-2ubuntu14.4_amd64.deb  playbook  shim-signed_1.51.3+15.7-0ubuntu1_amd64.deb
root@ub22:~# mkdir shim
root@ub22:~# dpkg -x shim-signed_1.51.3+15.7-0ubuntu1_amd64.deb shim
root@ub22:~# ll shim/usr/lib/shim/
total 4692
drwxr-xr-x 3 root root   4096 Jan 31  2023 ./
drwxr-xr-x 3 root root     18 Jan 31  2023 ../
-rw-r--r-- 1 root root    108 Jan 31  2023 BOOTX64.CSV
-rw-r--r-- 1 root root  88296 Jan 28  2023 fbx64.efi
-rwxr-xr-x 1 root root   1622 Jan 31  2023 is-not-revoked*
-rw-r--r-- 1 root root 860824 Jan 28  2023 mmx64.efi
drwxr-xr-x 2 root root     25 Jan 31  2023 mok/
-rw-r--r-- 1 root root 950891 Jan 31  2023 shimx64.efi
-rw-r--r-- 1 root root 962400 Jan 31  2023 shimx64.efi.dualsigned
-rw-r--r-- 1 root root 960472 Jan 31  2023 shimx64.efi.signed.latest
-rw-r--r-- 1 root root 955656 Jan 31  2023 shimx64.efi.signed.previous

root@ub22:~# scp shim/usr/lib/shim/shimx64.efi.signed.latest  192.168.1.3:/var/lib/tftpboot/grub/bootx64.efi
```
# /etc/dhcp/dhcpd .conf
## ipxe
        if exists user-class and option user-class = "iPXE" {
                filename "ipxe/ipxe.cfg";
        } elsif option client-architecture = 00:00 {
                filename "ipxe/undionly.kpxe";
        } else {
                filename "ipxe/ipxe.efi";
        }
/var/lib/tftpboot/ipxe/ipxe.cfg
![[Pasted image 20240420065630.png]]
ub，db不能装
## filename "grub/grubx64.efi";
## filename "grub/bootx64.efi";
以上两个都行，结果一样，bootx64.efi又加载了grubx64.efi，所以直接用grubx64.efi就行
/var/lib/tftpboot/grub/grub.cfg
![[Pasted image 20240419155421.png]]
![[Pasted image 20240419161716.png]]

Debian和co6无法安装

## filename "BOOTX64.efi";
可以兼容co6，因为BOOTX64.efi就是co6提供的

/var/lib/tftpboot/efidefault
![[Pasted image 20240419155046.png]]
![[Pasted image 20240419155100.png]]
这里等待时间较久

红帽系列都能装
ub，db不能装

## filename "uefi/shimx64.efi"; 这个不能用，不启动
![[Pasted image 20240420184125.png]]

option client-architecture code 93 = unsigned integer 16;
option arch code 93 = unsigned integer 16; # RFC4578
如果下面用：
if option arch = 00:07 or option arch = 00:09 {
        filename "uefi/BOOTX64.efi";
           filename "grub/grubx64.efi";
} else {
        filename "gpxelinux.0";
}
只能用option arch code 93 = unsigned integer 16; # RFC4578
否则，systemctl restart httpd tftp dhcpd会报错！

##  if option arch = 00:07 or option arch = 00:09
如果选项arch = 00:07或选项arch = 00:09，如果最终设备是基于UEFI的主机，请使用bootnetx64.efi文件启动设备，或使用不是基于UEFI的其他计算机启动用pxelinux.0文件。 这些机器通常是较旧的BIOS系统




