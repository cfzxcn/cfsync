之前先装了ub22-server，后装了win10双系统，现在要把ub22-server改为ub22-desktop
# 再装ub22 desktop
![[Pasted image 20240617194641.png]]
![[Pasted image 20240617194719.png]]
![[Pasted image 20240617194759.png]]
![[Pasted image 20240617194933.png]]

![[Pasted image 20240617200531.png]]
![[Pasted image 20240617200701.png]]
安装启动引导器的设备下没有EFI的那个区，所以只能选择/dev/nvme0n1
![[Pasted image 20240617202327.png]]
安装后重启自动就有了菜单，并且可选择win10启动
lsblk查看
![[Pasted image 20240617210033.png]]
root@cf:~# ls /media/cf/2d3deba1-773c-407e-9727-2a391f48e66d/
ub.txt
![[Pasted image 20240617205154.png]]
# 要解决的
## 1、默认安装后没有openssh-server，所以ssh无法连接
apt install openssh-server tree vim net-tools



