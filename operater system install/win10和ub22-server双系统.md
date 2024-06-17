# 配置vm虚拟机
![[Pasted image 20240615200321.png]] ![[Pasted image 20240615200406.png]]
![[Pasted image 20240615200513.png]] ![[Pasted image 20240615200548.png]]
![[Pasted image 20240615201020.png]] 
![[Pasted image 20240615201106.png]]
再次“编辑虚拟机设置”
确定，选项---高级，是uefi固件
再添加一块16T硬盘：硬件---添加---硬盘---磁盘类型选择SATA（因为模拟的第二块硬盘是机械硬盘），大小为16T（16384G）,
![[Pasted image 20240615201841.png]]
但vm给出如上提示，即可模拟的虚拟磁盘大小最大为：8T（8192G），所以改为：
![[Pasted image 20240615202155.png]]
最后结果：
![[Pasted image 20240615202244.png]]

# 用U盘启动虚拟机
## vm中添加U盘
需以管理员身份运行vmware
虚拟机设置—硬件—添加—硬盘—下一步—NVMe（V）--下一步—使用物理磁盘---
![[Pasted image 20240615203320.png]]
![[Pasted image 20240615203449.png]]
## 设置虚拟机U盘启动
电源---打开电源时进入固件（F）
![[Pasted image 20240615203642.png]]
![[Pasted image 20240615203712.png]]
# 用U盘安装win10
![[Pasted image 20240615203936.png]]
![[Pasted image 20240615204008.png]]
![[Pasted image 20240615204050.png]]
![[Pasted image 20240615211432.png]]
为win10的c盘划了1T，即：1024×1024 = 1048576M
![[Pasted image 20240615211444.png]]
划分完为：
![[Pasted image 20240615211708.png]]
下一页，开始安装windows
![[Pasted image 20240615212703.png]]
未识别出internet，因为在vm的虚拟网络编辑器中未选择：使用本地DHCP服务将IP地址分配给虚拟机
![[Pasted image 20240615213310.png]]
![[Pasted image 20240615213230.png]]
![[Pasted image 20240615213421.png]]
不输入密码（可留空），下一页
安装后--磁盘管理，提示：
![[Pasted image 20240615214329.png]]
目前资源管理器中只能看到1T的C盘，win10安装完成
## 重启win10直接进入bios，准备安装ub22
### 方法1：
单击“开始 -> 电源”按钮，按住 Shift 键，然后单击“重启”命令
### 方法2：
按Win+R 组合键，在“运行”对话框，输入并回车执行 shutdown -r -o
疑难解答--高级选项---UEFI固件设置
![[Pasted image 20240615220402.png]]
![[Pasted image 20240615220512.png]]
仍然选择U盘启动
![[Pasted image 20240615220607.png]]
# 用U盘安装ub22
![[Pasted image 20240615220651.png]]
清华源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
腾讯源：https://mirrors.cloud.tencent.com/ubuntu/
## 分区选择一
注：不要在ssd硬盘上用完所有空间，留一部分来安装win10，第二块sas盘可作为数据盘。这样做的原因在于：电脑开机时会自动在系统盘所在硬盘搜索启动项以启动系统，为使win10启动项也能被搜索到，所以也把win10安装在ssd硬盘上。如果空间紧张，可在ssd盘分出200M的空白分区用来安装ubuntu的启动项，然后再在另一块硬盘选择最后一个盘分配空间。

![[Pasted image 20240615233401.png]]
![[Pasted image 20240615233447.png]]
## Bios设置Ubuntu在Windows之前启动
## 将 Windows 10 添加到 GRUB 启动菜单：
```sh
root@cf:~# egrep -v '^($|#)' /etc/default/grub
GRUB_DISABLE_OS_PROBER=false     # 新增，Grub 2.06 不会自动探测其他操作系统安装并将其添加到启动菜单中，但将此行添加到 /etc/default/grub 文件中将解决该问题
#GRUB_DEFAULT=0   # 修改GRUB_DEFAULT的值来调整启动时，默认系统选择  0:ubuntu  2 win
GRUB_DEFAULT=saved    # 修改
GRUB_SAVEDEFAULT=true   # 新增
GRUB_TIMEOUT_STYLE=hidden   # 是否显示倒计时，hidden的属性表示不会显示倒计时
GRUB_TIMEOUT=10   # 修改，表示10s
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=""
GRUB_GFXMODE=auto   # 修改，屏幕的显示像素

root@cf:~# update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-112-generic
Found initrd image: /boot/initrd.img-5.15.0-112-generic
Warning: os-prober will be executed to detect other bootable partitions.
Its output will be used to detect bootable binaries on them and create new boot entries.
Found Windows Boot Manager on /dev/nvme0n1p1@/EFI/Microsoft/Boot/bootmgfw.efi
Adding boot menu entry for UEFI Firmware Settings ...
done

```
## 在Ubuntu中添加重启自动进入Windows系统功能
```sh
sudo grub-reboot 2  
sudo reboot
```
可以将其保存为`rebootwin.sh` 脚本文件并添加到`PATH`环境变量，然后终端输入`rebootwin`即可实现重启自动进入Windows系统。需要注意的是Windows菜单默认为第三个，所以为2，Ubuntu默认为第一个，为0
## 更新 Ubuntu 的linux驱动程序
有可能完成双系统安装后进入 Ubuntu 22 发现黑屏，左上角光标一直在闪，这很可能是显卡驱动程序不匹配导致的，可以无界面登录，更新驱动程序解决该问题。
按下 CTRL + ALT + F2 登录系统，执行自动安装驱动的命令：sudo ubuntu-drivers autoinstall
直接使用Ubuntu的命令行模式安装：输入`sudo bash 驱动文件名`进行安装
这里会提示系统默认使用了第三方驱动nouveau（关于nouveau可以自行百度一下），安装程序询问是否帮你创建模块文件来停用nouveau，这里选择左边的“yes”。
之后安装程序会退出
重启后我们直接登录回到图形界面，再次输入sudo bash 驱动文件名进入安装程序会提示未安装gcc包。
退出后，终端输入sudo apt install build-essential安装。

最后再一次输入sudo bash 驱动文件名打开安装程序进行安装，这时就可以正常安装了。
如显示：Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up →选择“yes”
如无问题，安装结束。

现在能正常显示显卡了！
我们在终端输入`nvidia smi`可以正常显示GPU信息了！


