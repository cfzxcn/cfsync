### 拷贝内核镜像文件：initrd和vmlinuz
1. 挂载 ISO 文件，注：只有live版本才能支持subiquity
```sh
mount /dev/sr1 /media/
或：
mount ubuntu-20.04-live-server-amd64.iso /media/

```
2. 拷贝
```sh
[root@co7hw ~]# cp /media/casper/initrd /var/lib/tftpboot/ub20/
[root@co7hw ~]# cp /media/casper/vmlinuz /var/lib/tftpboot/ub20/
```
### 拷贝菜单文件：grub.cfg和font.pf2
```sh
mkdir /var/lib/tftpboot/grub
cp /media/boot/grub/grub.cfg /var/lib/tftpboot/grub/
cp /media/boot/grub/font.pf2 /var/lib/tftpboot/grub/
```
### 修改菜单文件：/var/lib/tftpboot/grub/grub.cfg
https://docs.cloud-init.io/en/latest/reference/datasources/nocloud.html#datasource-nocloud
例：ds=nocloud;s=http://10.42.42.42/cloud-init/configs/
#### 允许的键是：
- `h`或者`local-hostname` 
- `i`或者`instance-id`
- `s`或者`seedfrom`
有效值`seedfrom`包括：
#### 文件系统
`/`以或开头的文件系统路径`file://`，指向包含文件的目录：`user-data`、`meta-data`和 （可选） （需要`vendor-data`尾随）`/`
#### HTTP 服务器
`http`或URL （需要`https`尾随）`/`
#### 文件格式
这些用户数据和元数据文件需要作为同一基本 URL 中的单独文件：
/user-data
/meta-data
两个文件都必须存在才能被视为有效的种子 ISO。
该`user-data`文件使用[用户数据格式](https://docs.cloud-init.io/en/latest/explanation/format.html#user-data-formats)， `meta-data`是一个 YAML 格式的文件，代表您在 EC2 元数据服务中找到的内容。
您还可以选择在同一基本 URL 上提供符合[用户数据格式](https://docs.cloud-init.io/en/latest/explanation/format.html#user-data-formats)的供应商数据文件 ：/vendor-data
```yaml
linux   /kernel/ub20/vmlinuz ip=dhcp url=http://192.168.1.3/iso/ub20/ubuntu-20.04-live-server-amd64.iso autoinstall ds="nocloud-net;s=http://192.168.1.3/ks/ub20/" cloud-config-url=/dev/null quiet ---
#  ip=dhcp必须要加，否则客户端启动后会报：wget: cat't connect to remote host (192.168.1.3): Network is unreachable

#  linux   /kernel/ub20/vmlinuz，指：/var/lib/tftpboot/kernel/ub20/vmlinuz
#  ds=nocloud-net\;s  由于 UEFI 启动使用grub，它将 ; 识别为了特殊字符，所以要在 ; 前加 \ 转义或使用单双引号
```
### /var/www/html/目录结构及配置
- 对于生产环境设置适当的权限，只需读权限
- /var/www/html/也就是http://192.168.1.3/
- autoinstall目录/var/www/html/ks/ub20存放参数自动配置文件，`user-data`、`meta-data` 是cloud-init 要求的文件名
- 创建空文件：`touch /var/www/html/autoinstall/meta-data`，`meta-data` 无需修改
```sh
/var/www/html/
├── ks
│     ├── alma85.cfg
│     └── anolis7.9.cfg
│     └── co7.cfg
│     └── co7.graphical.cfg
│     └── rocky85.cfg
│     ......
 |      |── ub20
│            ├── meta-data
│            └── user-data
├── iso
      ├── anolis7.9
      ├── centos7
      └── ub20
       |      └── ubuntu-20.04-live-server-amd64.iso
      └── ub22
            └── ubuntu-22.04-live-server-amd64.iso
```            
### 配置/var/www/html/ks/ub20/user-data
确保目录 /var/www/html/ks/ub20/ 下的文件对所有人可读：
chmod -R a+r /var/www/html/ks/ub20/
>注：在准备 cloud.init config 前。可以先手动安装一次 ubuntu 20.04，在 /var/log/installer/ 目录下会生成一个 autoinstall-user-data ，这是基于当前系统的应答文件，可以它为基础，根据实际情况进行修改。  /var/log/installer/autoinstall-user-data
#### 官方给出的cloud-init 配置：
```yaml
mkdir -p ~/www
cd ~/www
cat > user-data << 'EOF'
#cloud-config
autoinstall:
  version: 1
  identity:
    hostname: ubuntu-server
    password: "$6$exDY1mhS4KUYCE/2$zmn9ToZwTKLhCw.b4/b.ZRTIZM30JZ4QrOQ2aOXJ8yk96xpcCof0kxKwuX1kqLG/ygbJ1f8wxED22bTL4F46P0"
    username: ubuntu
EOF
#  加密后的密码是ubuntu
touch meta-data
```
#### 测试用的user-data
可安装成功，但很慢，且alt+F2可看到报错：
```sh
[root@co7hw ~]# cat /var/www/html/ks/ub20/user-data 
#cloud-config
autoinstall:
  version: 1
  apt:
    primary:
    - arches: [default]
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
  identity:
    hostname: ub20
    password: "$6$exDY1mhS4KUYCE/2$zmn9ToZwTKLhCw.b4/b.ZRTIZM30JZ4QrOQ2aOXJ8yk96xpcCof0kxKwuX1kqLG/ygbJ1f8wxED22bTL4F46P0"
    username: cf       
  locale: en_US
  ssh:
    allow-pw: true
    authorized-keys: []
    install-server: true
#  口令是：ubuntu
```
安装输出卡在installing security updates很久，禁用网卡配置中的 DHCP，设置dhcp4:no可解决此问题，下方有详细说明
![[Pasted image 20240405035017.png]]
alt+F2后，tail -f /var/log/syslog，可看到报错：
![[Pasted image 20240405034926.png]]
#### 密码可用如下方式生成：
```bash
root@ub20:~/playbook# mkpasswd -m sha-512 'cf'
$6$twj/RYN2kVTvg$1iEOYPMV.0fe.mbvq4f03totG5TIYPeQqRisz8/rBLoFzuOJSt8NLVMZIviDe2F5mrgY0/mhQWjflyOVad0TV1

#  cf是要生成密码的口令；默认ub中mkpasswd未安装，所以：
#  root@ub20:~/playbook# apt install whois，whois在co7中安装也没有mkpasswd
或：
root@ub20:~/playbook# printf "cf" | mkpasswd -s -m sha-512
$6$j3zhtbseY1$PzvMxqolXwmaevYzTUMNmch8y1eHd3apSWdZuXvUgBKceP0Ny2JIKSw1J0EmsiUvaCxLyd9blScQ8MA7e9j3z.

printf "cf" | mkpasswd -s -m md5
$1$lbN87E/s$rVvTNACJ4VzH4kFkWGevO.
```
#### 更多选项说明
- 使用`cloud-config-url=/dev/null`内核参数将防止`cloud-init`不必要地下载 iso，从而[减少所需的内存](https://discourse.ubuntu.com/t/ubuntu-20-04-autoinstall-with-2g-ram/21711/2)、减少网络流量并加快启动时间。
- 生成的`/var/log/installer/autoinstall-user-data`文件会被以下方式被破坏
    - 没有`version`属性，导致验证失败。我添加了属性
    - 该`network`部分需要另一层嵌套。配置参考中提到了这个错误
    - `preserve`需要将每个项目的属性设置`storage` `config`为**false**。否则_curtin_不会安装在空白磁盘上
    - 该`keyboard`属性`toggle`设置为 null，这导致验证失败。我只是删除了该属性
- 当_curtin_安装在 UEFI 设备上时，它会重新排序引导顺序，以便当前引导选项位于列表中的第一个。结果是网络启动成为下次重新启动时的第一个选项。因此，当安装完成并重新启动时......您最终会再次进入 PXE 环境，而不是从磁盘启动。我发现了一个**未记录的** _科廷_选项`reorder_uefi`。幸运的是，_subiquity_恰好将这个配置传递给_curtin_
- 配置`apt`选项`geoip`似乎不起作用。总是有 geoip 请求的日志
- 使用人类可读的分区大小值（例如`size: 512M`）会导致大小被存储为浮点数，从而导致在以百分比调整 LVM 卷大小时出现错误。避免人类可读的值似乎可以解决这个问题
```yaml
network:
    network:
      version: 2
      ethernets:
        eth0:
          dhcp4: no
          dhcp6: no

#  注意：
1、上面的network是两层。20.04 有一个错误，需要进行这样的设置，而 20.10 没有该错误，但支持此作为向后兼容功能
2、服务器将没有网络（因为禁用了 DHCP，dhcp4:no），因此它将在安装过程中跳过自动更新（以加快测试速度）。实测设置 dhcp4: no 后，安装速度飞快，且安装后的网卡配置为：
root@ub20:~# cat /etc/netplan/00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      critical: true
      dhcp-identifier: mac
      dhcp4: true
      nameservers:
        addresses:
        - 180.76.76.76
        - 223.5.5.5
        search:
        - sheying001.com
  version: 2
可看出，自动把dhcp4设置为了 true，且只有一层network

ub22也能这样加速安装，但装完并未把dhcp4自动设置为true
root@ub22:~# cat /etc/netplan/00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      critical: true
      dhcp-identifier: mac
      dhcp4: false
      dhcp6: false
  version: 2
  #  所以ub22只能手动设置为dhcp4: true，这是个问题，因为如果是这样，生产环境中得一台台去启用
```
#### 分区
![[Pasted image 20240405132951.png]]
cf@ub20:~$ lsblk 
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 27.1M  1 loop /snap/snapd/7264
loop1                       7:1    0   55M  1 loop /snap/core18/1705
loop2                       7:2    0   69M  1 loop /snap/lxd/14804
sda                         8:0    0  100G  0 disk 
├─sda1                      8:1    0  512M  0 part /boot/efi
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0 98.5G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0    4G  0 lvm  /
***
   \- {device: disk-sda, size: 536870912, wipe: superblock, flag: boot, number: 1,
     preserve: false, grub_device: true, **type: partition**, ==id: partition-0==}
   \- {fstype: fat32, volume: ==partition-0==, preserve: false, type: format, ==id: format-0==}

   \- {device: ==format-0==, path: /boot/efi, type: mount, id: mount-0}
***
   \- {device: disk-sda, size: 1073741824, wipe: superblock, flag: '', number: 2,
     preserve: false, **type: partition**, ==id: partition-1==}
   \- {fstype: ext4, volume: ==partition-1,== preserve: false, type: format, ==id: format-1==}

   \- {device: ==format-1==, path: /boot, type: mount, id: mount-1}
***
  \ - {device: disk-sda, size: 105761472512, wipe: superblock, flag: '', number: 3,
      preserve: false, **type: partition**, i==d: partition-2==}
  \ - name: ubuntu-vg
      devices: [partition-2]
      preserve: false
      type: lvm_volgroup
      id: lvm_volgroup-0

可看出，以上三个type都是 partition
***
   \- {name: ubuntu-lv, volgroup: lvm_volgroup-0, size: 4294967296B, preserve: false,
      type: lvm_partition, ==id: lvm_partition-0==}
  \ - {fstype: ext4, volume: ==lvm_partition-0==, preserve: false, type: format, ==id: format-2==}
  \ - {device: ==format-2==, path: /, type: mount, id: mount-2}
可看出，这是根分区
***
cf@ub20:~$ echo $[536870912/1024/1024]
512M
cf@ub20:~$ echo $[1073741824/1024/1024]
1024M
cf@ub20:~$ echo $[105761472512/1024/1024]
100862M
cf@ub20:~$ echo $[105761472512/1024/1024/1024]
98G
cf@ub20:~$ echo $[4294967296/1024/1024]
4096M
***
### 配置/etc/dhcp/dhcpd.conf
指定启动时的引导文件
### 排错
#### 在server上执行：
tail /var/log/httpd/access_log
tail /var/log/httpd/error_log
journalctl | grep autoinstall
journalctl -u dhcpd.service
#### 在client上执行：
[ctrl+]alt+F2 进入命令行，

tail /var/log/installer/subiquity-curtin-install.conf

, path: etc/default/keyboard, permissions: 4203etc_machine_id: {content: '7b53e4dc636342d6bf28681ed885f763, 
path: etc/machine-id
, permissions: 2923etc_netplan_installer: {content: "# This is the network config written by 'subiquity'n^network:\n ethernets:nens33:^ncritical: true\ndhcp-ident if ier:\mac\ndhcp4: true\nnameservers:\naddresses:\n-<\180.76.76.76\n-223.5.5.5nsearch:^n- sheying001.comn\version: 2<n, 
path: etc/netplan/00-installer-config.yaml
md5check:{content:"{\n\"checksum_missmatch\":[\n],\n<"result\":\"pass\"\n\3\n', 
path: var/log/installer/casper-md5check.json
, permissions: 4203media_info: {content: Ubuntu-Server 20.04 LTs "Focal Fossa" - Release amd64 (20200423),
path: var/log/installer/media-info
, permissions: 4203nonet: {content: 'network: {config: disabled}
path: etc/cloud/cloud.cfg.d/subiquity-disable-cloudinit-networking.cfg

dmesg | grep 'Command line'
\# 将显示完整的`ds`参数是否正在被传递，或者是否仅传递到`;`

journalctl | grep autoinstall

journalctl
tail -f /var/log/syslog
\#  debian可用
#### 待解决：
1. 网卡是ens33，如何改为eth0
2. 设置了Swap分区为2G，但free -h查询并非2G，而是5.8G，原因未知！
3. systemctl get-default结果为：graphical.target，但未安装图形化，可能需要使用desktop的镜像

![[Pasted image 20240407232148.png]]
# 已解决问题
## run_unattended_upgrades: downloading and installing security updates
pxe 安装 ubuntu 20.04的过程中，出现以下提示，看出是因为安装安全更新的问题，并且一直等待，不会自动完成安装，虽然系统安装已完成，此时重启也可正常使用，但有点别扭，并不完美
![[Pasted image 20240429201012.png]]
![[Pasted image 20240429200938.png]]
### 解决方法
通过禁用网络来禁用自动更新
```
network:
  ethernets: {}
  version: 2
```
但pxe安装完，仍会自动配置网卡并启用，并不会没有ip，目前可较完美的解决此问题！
注：22.04没有此问题







