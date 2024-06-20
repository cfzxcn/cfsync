![[Pasted image 20240619215134.png]]
之前先装了ub22-server，后装了win10双系统，现在要把ub22-server改为ub22-desktop
# 所以再装ub22 desktop
说明：
- 要装win10+ub22-desktop双系统，先装win，再装ub22-desktop双系统，这样装完直接启动默认就是ub22，自动出现启动菜单，并可以选择win启动
- 要装win10+ub22-server双系统，先装server，再装win，这样装完直接启动默认是win，可进入server去配置启动菜单，并在bios中指定server为第一启动项，这样启动后才出现启动菜单，可以选择win启动。
- 先装win，再装server，是否可行，我没测试过。不过先装win，再装ub22-desktop，ub22在安装过程中分区时，可以选择：“其他选项”，以与windows Boot Manager共存。而先装win，再装server，安装过程中分区时没有：“其他选项”，所以不确定是否能安装成功，也许是可以的，但我没往下继续测试

![[Pasted image 20240617194641.png]]
![[Pasted image 20240617194719.png]]
![[Pasted image 20240617194759.png]]
![[Pasted image 20240617194933.png]]

![[Pasted image 20240617200531.png]]
下次选择成ext4试下，xfs在真实服务器安装后启动时，会有error提示
![[Pasted image 20240617200701.png]]
安装启动引导器的设备下没有EFI的那个区，所以只能选择/dev/nvme0n1
![[Pasted image 20240617202327.png]]
安装后重启自动就有了菜单，并且可选择win10启动
lsblk查看
![[Pasted image 20240617210033.png]]
root@cf:~# ls /media/cf/2d3deba1-773c-407e-9727-2a391f48e66d/
ub.txt
![[Pasted image 20240617205154.png]]
# 安装系统后要做
## 1、默认安装后没有openssh-server，所以ssh无法连接
```sh
apt update
apt install openssh-server tree vim net-tools lrzsz
#  默认ub22不能ssh连接22端口，因为没装openssh-serve
```
# NVIDIA驱动安装
## 查看GPU型号
lspci | grep -i nvidia
## 卸载原有驱动
sudo apt-get remove --purge nvidia*   # 或者nvidia-*
## 关闭图形界面进入tty模式
```sh
sudo telinit 3
#  sudo telinit 5或sudo init 5可重新打开图形界面
export LANGUAGE="UTF-8"
#  进入tty模式如果不是英语系统会出现乱码，export LANG="UTF-8" 或LANG=C不管用
#  可随意输入一个不存在的命令来测试系统是否出现乱码
apt install make gcc g++
#  必须先装make，否则安装Nvidia驱动时会报错
sudo bash NVIDIA-Linux-x86_64-550.90.07.run
```
![[Pasted image 20240620013638.png]]
上图说可以通过ub的software & updates---Additional Drivers来安装Nvidia的驱动，我们不用ub的，所以：Continue installation
![[Pasted image 20240620014501.png]]
说 nouveau驱动现在正被系统使用，此驱动不兼容Nvidia驱动，必须在继续安装前禁用，所以：
## 禁用系统自带显卡驱动nouveau
Ubuntu22.04默认使用的显卡驱动是由Linux一众开发者自己写的nouveau，实际使用下来非常差劲。。。只要进入火狐/Chrome等应用一定卡死，即使在桌面上没做什么操作有时候也会卡死，所以要卸载掉
以后安装Nvidia驱动过程中会自动生成两个文件，会自动禁用nouveau的
```sh
sudo vim /etc/modprobe.d/blacklist.conf
在打开的文件末尾加上：
blacklist nouveau

lsmod | grep nouveau
#  重启后在终端输入如下，没有任何输出表示屏蔽成功
```
## 在终端输入如下更新，更新结束后重启电脑（必须）
sudo update-initramfs –u

sudo bash NVIDIA-Linux-x86_64-550.90.07.run
重新安装

## 测试Nvidia驱动是否安装成功
```sh
nvidia-smi
watch -n 1 -d nvidia-smi
#  实时监测
nvidia-settings
#  调出设置界面
```

## 安装Nvidia驱动可能出现的问题
### 死机，卡在logo没法进入
可以在bios里面把双显卡模式改成独显模式
**解决办法（未实际测试）：**
1. （如果已经死机，只能强制关机）**开机**
2. （在选择系统的界面，如上图）**上下键选择ubuntu高级选项，回车**
3. （在出现的两个模式中）**选择recovery（恢复）模式，回车**
4. （在出现的多个选项中）**选择grub，回车**，接着会看到一行行代码自动运行
![[4e6da4b0ba248e78a05d5e4c82b620dc_watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGVIb25naHU=,size_16,color_FFFFFF,t_70.png]]
5. （重新回到多个选项的界面）**选择resume，回车**
但是每次开机这样都可以进入系统，但是很麻烦，所以下面我们进行永久修改（前提：已经通过上面的方式进入了系统）：
1. **修改/etc/default/grub文件：**
在Ubuntu系统内，打开终端 terminal，在符号“$”后输入命令：  sudo gedit /etc/default/grub，将其中的“quiet splash”修改为“quiet splash nomodeset”并保存;
2. 更新修改完的grub：sudo update-grub
之后回车即可，独立显卡导致死机的问题也就被完美解决了，以后进系统正常进就行



