# httpd
chmod -R +r /var/www/html/ks/
## 解决AH00558报错
systemctl status httpd.service或httpd -t时会有：AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::c72a:62f2:2e9f:5ce2. Set the 'ServerName' directive globally to suppress this message的提示。不解决也行，不影响使用
解决方法：vim /etc/httpd/conf/httpd.conf，95下，添加：ServerName localhost:80。systemctl restart httpd即可
## 挂载iso镜像并拷贝Linux 内核、虚拟根文件
```sh
mkdir -p /var/www/html/{ks,sh,iso/{alma85,anolis7.9,co6,co7,db10,db11,db125,db9,kylin-Server-V10-SP3,openEuler-22.03-LTS-SP3,ub20,ub22,uos20-1060e}}

mount /dev/sr3 /var/www/html/iso/kylin-Server-V10-SP3/
mount /var/www/html/iso/centos7/CentOS-7-x86_64-DVD-2009.iso /var/www/html/iso/co7
......
cd /var/www/html/iso/co7/images/pxeboot/

cp vmlinuz /var/lib/tftpboot/kernel/co7
#  复制 Linux系统的内核文件 到TFTP目录下
cp initrd.img /var/lib/tftpboot/kernel/co7
#  复制 虚拟根文件（linux引导加载模块）到TFTP目录下
```
# nfs
```sh
yum -y install nfs-utils
systemctl enable --now nfs-server;systemctl status nfs-server
mkdir /var/www/html/iso/rocky85
mount /dev/sr2 /var/www/html/iso/rocky85
echo '/var/www/html/iso/rocky85 *(rw,sync)' >> /etc/exports
#  这样就不用建立专门的nfs目录了，直接用httpd的。注意：必须指定rocky85子目录，如指定成：/var/www/html *(rw,sync)，是不行的
echo ' /var/www/html/ks *(rw,sync,no_root_squash) ' >> /etc/exports; exportfs -r
exportfs -r
#  使nfs生效

[root@pxe kernel]# showmount -e localhost
Export list for localhost:
/var/www/html/iso/rocky85 *
[root@pxe kernel]# exportfs
/var/www/html/iso/rocky85
		<world>
# showmount -e localhost和exportfs有输出表明nfs生效

grub.cfg中添加如下：
linux  /kernel/co7/vmlinuz inst.repo="nfs:${IP}:/var/www/html/iso/centos7/CentOS-7-x86_64-DVD-2009.iso" inst.ks="nfs:${IP}:/var/www/html/ks/co7.nfs.cfg"
#  第一种方法不需要挂载iso，但要把iso拷贝到硬盘上
或
linux  /kernel/co7/vmlinuz inst.repo="nfs:${IP}:/var/www/html/iso/co7/" inst.ks="nfs:${IP}:/var/www/html/ks/co7.nfs.cfg"
#  第二种方法需要挂载iso
#  两种方法均可，和co7.nfs.cfg中要对应，但推荐第二种方法，第一种方法重启有几个error，过的太快没看清

co7.nfs.cfg中添加如下：
nfs --server=192.168.1.3 --dir=/var/www/html/iso/centos7/CentOS-7-x86_64-DVD-2009.iso
或
nfs --server=192.168.1.3 --dir=/var/www/html/iso/co7/
#  两种方法均可，和grub.cfg中要对应，推荐第二种方法
```
## nfs问题
```sh
[root@pxe ~]# umount /var/www/html/iso/rocky9
umount: /var/www/html/iso/rocky9：目标忙。
        (有些情况下通过 lsof(8) 或 fuser(1) 可以
         找到有关使用该设备的进程的有用信息)
#  用nfs有时会出现无法卸载的情况，且用推荐的lsof或fuser命令也无法卸载，所以推荐用httpd
```



