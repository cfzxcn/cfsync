---
date: 2024-03-19
tags:
  - gpt
  - lvm
  - u盘安装
---
# 下载
```sh
G:\iso\linuxubuntu-22.04.4-desktop-amd64.iso.torrent
```
用qBittorrent下载
# vm设置
^7319a3
## 添加U盘
vm以管理员身份启动---虚拟机设置---硬件---添加---硬盘---scsi（推荐）---使用物理磁盘---选择最后一个设备（physicaDrive3），使用整个磁盘
## 使用u盘
虚拟机设置---硬件---usb控制器---兼容性---3.1
## 使用gpt
虚拟机设置---选项---高级---UEFI
# 安装
## 电源---打开电源时进入固件
选择：EFI---VMware Virtual SCSI Hard Driver (1.0)---然后就出现了ventoy的界面---选择ubuntu---* Try or Install Ubuntu
![[Pasted image 20240319010324.png]]
![[Pasted image 20240319011301.png]]
![[Pasted image 20240319011428.png]]
















