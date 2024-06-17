# [FAILED] failed to start switch root
![[Pasted image 20240508165033.png]]
解决方法：grub.cfg中url改成inst.repo
# Kickstart file /run/install/ks.cfg is missing
![[Pasted image 20240510055034.png]]
原因：ks文件语法错误，services --disabled="rpcbind.target，最后少了个分号，这么个小问题，排查了三天，郁闷！
# pane is dead
原因：感觉大部分是因为ks语法问题导致
# 虚拟机已经安装了ub20，再安装co7，如下图提示
在无外网环境测试安装
menuentry --hotkey=3 '3  CentOS 7.9  en_US  GPT  NoLvm  Text
![[Pasted image 20240522041858.png]]
clearpart --all --initlabel --drives=sda
IOExcention: Partition(s) 3 on /dev/sda have been written, but we have been unable to inform the kernel of the change, probably because it/ they are in use. 
异常:/dev/sda上的分区3已经被写入，但是我们无法通知内核更改，可能是因为它们正在被使用。
As a result, the old partition(s) uill remain in use. 
因此，旧分区将继续使用。
You should reboot now before making furtherchanges
在进行进一步更改之前，您应该重新启动
解决方案：添加：ignoredisk --only-use=sda
# pxe无外网时安装rocky9报错：设置基础软件仓库时出错
![[Pasted image 20240528040529.png]]
解决方法：将ks文件中用于访问外网的：repo --name="epel" --baseurl="http://mirrors.cloud.tencent.com/epel/9/Everything/x86_64/" --install，注释掉即可
但co7也有这样的设置，但可成功安装，并不受影响，看来rhel8/9版本这方面有的变化
# httpd安装rocky85报错：Error setting up software source
而nfs安装rocky85成功，nfs安装的ks文件中只用：nfs --server=192.168.1.3 --dir=/var/www/html/iso/rocky85 即可，而httpd安装的ks文件中用
```
# repo --name="rocky85" --baseurl="http://192.168.1.3/iso/rocky85/"是不行的，因为是8版本，所以要用下面两句：
repo --name="AppStream" --baseurl="http://192.168.1.3/iso/rocky85/AppStream"
repo --name="BaseOS" --baseurl="http://192.168.1.3/iso/rocky85/BaseOS"
```
并且要注释：#repo --name="epel" --baseurl="http://mirrors.cloud.tencent.com/epel/8/Everything/x86_64/" --install






