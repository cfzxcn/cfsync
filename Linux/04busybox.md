## Busybox介绍
Busybox 最初是由 Bruce Perens 在 1996 年为 Debian GNU/Linux 安装盘编写的。其目标是在一张软盘(存储空间只有1MB多)上创建一个GNU/Linux 系统，可以用作安装盘和急救盘

Busybox 是一个开源项目，遵循GPL v2协议。Busybox将众多的UNIX命令集合进一个很小的可执行程序中，可以用来替代GNU fileutils、shellutils 等工具集。Busybox中各种命令与相应的GNU工具相比，所能提供的选项比较少，但是也足够一般的应用了。Busybox主要用于嵌入式系统

Busybox 是一个集成了三百多个最常用Linux命令和工具的软件。BusyBox 包含了一些简单的工具，例如ls、cat和echo等等，还包含了一些更大、更复杂的工具，例grep、find、mount以及telnet。有些人将 BusyBox 称为 Linux 工具里的瑞士军刀。简单的说BusyBox就好像是个大工具箱，它集成压缩了Linux 的许多工具和命令，也包含了 Android 系统的自带的shell

定制小型的Linux操作系统：linux内核+busybox

官方网站：https://busybox.net/
https://busybox.net/downloads/busybox-1.36.0.tar.bz2
## ## busybox使用
busybox 的编译过程与Linux内核的编译类似

busybox的使用有三种方式：
- busybox后直接跟命令，如 busybox ls
- 直接将busybox重命名，如 cp busybox tar
- 创建符号链接，如 ln -s busybox rm

**busybox的安装**
以上方法中，第三种方法最方便，但为busybox中每个命令都创建一个软链接，相当费事，busybox提供自动方法：busybox编译成功后，执行make install,则会产生一个_install目录，其中包含了busybox及每个命令的软链接
## busybox编译安装
1. yum -y install gcc gcc-c++ glibc glibc-devel make pcre pcredevel openssl openssl-devel systemd-devel zlib-devel glibc-static ncurses-devel
2. wget -N -P /opt https://busybox.net/downloads/busybox-1.36.0.tar.bz2 -C /opt
3. tar xvf busybox-1.36.0.tar.bz2 &&  cd busybox-*

4. make menuconfig 
\#  把busybox编译成静态二进制，不用共享库：
\#  Settings -->Build Options -->[\*] Build static binary (no shared libs)-->exit-->exit-->yes
\#  静态编译：把所有的库和二进制文件打包在一起，而不是分别独立，默认，编译时它们分别独立
![[Pasted image 20230924140209.png]]
![[Pasted image 20230924140542.png]]

5. make  && make install  \#  如果出错，执行make clean后，重新执行上面命令
**报错：**
miscutils/seedrng.c:45:24: 致命错误：sys/random.h：没有那个文件或目录
 \#include <sys/random.h>
                        ^
编译中断。
make\[1\]: *** [miscutils/seedrng.o] 错误 1
make: *** [miscutils] 错误 2






