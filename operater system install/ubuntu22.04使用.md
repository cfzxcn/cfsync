## xshell使用root登录
1. 设置root密码
2. 修改sshd_config
	1. vim /etc/ssh/sshd_config
	2. \#PermitRootLogin prohibit-password改为：PermitRootLogin yes
3. 重启ssh
	1. /etc/init.d/ssh restart  或：
	2. service ssh restart  或：
	3. systemctl restart ssh
## 设置密码
echo root:cf | chpasswd
==为root设置密码为cf，co中应该也能这样用==
echo cf | passwd --stdin root
==centos中设置密码的一般用法==
## 新主机init1
配置了静态ip，root密码，ssh允许用root登录，添加了别名，更改了命令提示符颜色，设置了主机名，更新了/etc/hosts
apt install autofs bind9-utils lrzsz net-tools sshpass tree git
==安装了如上常用软件==
## ub20.04没有/var/log/messages系统日志文件解决办法
1. 默认没有此文件，但有：tail /var/log/syslog
2. 如果要开启此文件，则：vim /etc/rsyslog.d/50-default.conf，可在34行下添加：
`*.info;mail.none;authpriv.none;cron.none        /var/log/messages`
3. 保存该配置文件，然后无需重启系统，只需要重启rsyslog这个服务即可
`systemctl restart rsyslog.service`
## 设置时区
timedatectl set-timezone Asia/Shanghai
reboot后也能生效

曹先生
15101665448
也可微信联系，微信号：ainncf
