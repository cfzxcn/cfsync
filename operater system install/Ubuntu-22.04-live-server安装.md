---
date: 2024-03-18
tags:
  - gpt
  - lvm
  - u盘安装
---
# vm配置
[[Ubuntu-22.04.4-deskto安装#^505b08]]
# 安装
![[Pasted image 20240319161842.png]]
上图如选择第三项，则进入：
![[Pasted image 20240319161931.png]]
上图选择第一项，返回安装界面
![[Pasted image 20240319162317.png]]
## 静态网卡配置
![[Pasted image 20240318212516.png]]
## 代理留空
![[Pasted image 20240318212845.png]]
## 改成国内镜像地址
![[Pasted image 20240318212945.png]]
阿里源：http://mirrors.aliyun.com/ubuntu/ 
网易源：http://mirrors.163.com/ubuntu/
清华源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
腾讯源：https://mirrors.cloud.tencent.com/ubuntu/
## 分区设置：gpt+lvm
### use an entire disk并选中lvm
![[Pasted image 20240318215155.png]]
下次试试先 reset，再全部手工分区
默认配置如下，sdb是U盘，不用管
![[Pasted image 20240318231024.png]]
### 调整/boot/efi分区为150M
默认是1.049G，我觉得太大有点浪费，所以调小些
![[Pasted image 20240318231351.png]]
![[Pasted image 20240318231419.png]]
### 调整/boot分区为1G
![[Pasted image 20240318231451.png]]
![[Pasted image 20240318231524.png]]
### 把剩余空间都划给根
如下图，执行完以上几步默认ubuntu-lv即/是48.727G，还有个free space是48.727G，想把这部分空间也划给/，所以先删除默认的/，再删除ubuntu-vg，这样就能把所有剩余空间都划给vg，再全部划给/。
![[Pasted image 20240318231803.png]]
*还有个free space是1.903G，经测试这部分空间无法删除，所以只能保留不划分，这个问题待以后解决，第二次新建一个虚拟机，把它划分给了/swap，估计不是swap虚拟内存，只是一个叫swap的普通分区，这个空间怎么利用好，这个问题待以后解决！*
![[Pasted image 20240318233458.png]]
#### 删除默认的/，即ubuntu-lv
![[Pasted image 20240318232114.png]]
#### 删除默认的ubuntu-vg
![[Pasted image 20240318232356.png]]
### 最后效果：
![[Pasted image 20240318233919.png]]

![[Pasted image 20240318234044.png]]
*忘记划分swap分区了，不过装完也能用，装完查看*
![[Pasted image 20240319002118.png]]
## name和pw都是cf
![[Pasted image 20240318234609.png]]
## ssh设置
![[Pasted image 20240318234715.png]]
## 软件选择
我啥都没选
![[Pasted image 20240318234807.png]]
## 重启并断开与u盘的连接
![[Pasted image 20240319000742.png]]

虚拟机上安装完毕，默认重启又开始安装，所以可以：
电源---打开电源时进入固件，选择：
EFI VMware Virtual SCSI Hard Drive (0,0)
启动，以后重启或开机就会直接进入系统了






