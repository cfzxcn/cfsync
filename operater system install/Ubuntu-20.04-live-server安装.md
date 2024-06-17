---
date: 2024-04-03
uefi:
  - lvm
  - gpt
  - uefi
  - 100G
  - en_US
---
# vm设置
## 使用gpt
虚拟机设置---选项---高级---UEFI
## 开始安装
![[Pasted image 20240403162153.png]]
![[Pasted image 20240403162814.png]]
![[Pasted image 20240403162857.png]]
代理：无
![[Pasted image 20240403162949.png]]
## 分区
![[Pasted image 20240403163116.png]]
默认如下：
![[Pasted image 20240403163303.png]]
### 更改/boot.efi大小为100M
![[Pasted image 20240403163414.png]]
![[Pasted image 20240403163455.png]]
### 删除/，因为默认只有4G
![[Pasted image 20240403163727.png]]
### 添加swap，默认没有
![[Pasted image 20240403163909.png]]
![[Pasted image 20240403163959.png]]
### 再添加根分区，所有容量都给它
![[Pasted image 20240403164056.png]]
![[Pasted image 20240403164149.png]]
### 更改/boot为xfs，默认为ext4
![[Pasted image 20240403164514.png]]
![[Pasted image 20240403164544.png]]
### 分区完的样子，默认有412M不可分区
![[Pasted image 20240403164826.png]]
## 用户名
![[Pasted image 20240403164925.png]]
安装后测试登陆用户名为：zx；主机名为：st；
查看：cat /var/log/installer/autoinstall-user-data
realname: cf, username: zx
## 软件使用默认，什么都不选
![[Pasted image 20240403165011.png]]
## 取消更新并重启
![[Pasted image 20240403170210.png]]




