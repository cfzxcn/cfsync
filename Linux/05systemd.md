![[Pasted image 20230917161409.png]]
![[Pasted image 20230917161316.png]]
***
## Linux 的启动
Linux 操作系统的启动首先从 BIOS 开始，然后由 Boot Loader 载入内核，并初始化内核。内核初始化的最后一步就是启动 init 进程。这个进程是系统的第一个进程，PID 为 1，又叫超级进程，也叫根进程。它负责产生其他所有用户进程。所有的进程都会被挂在这个进程下，如果这个进程退出了，那么所有的进程都被 kill 。如果一个子进程的父进程退了，那么这个子进程会被挂到 PID 1 下面。（注：PID 0 是内核的一部分，主要用于内进换页）
### SysV Init
PID 1 这个进程非常特殊，其主要就任务是把整个操作系统带入可操作的状态。比如：启动 UI – Shell 以便进行人机交互，或者进入 X 图形窗口。传统上，PID 1 和传统的 Unix System V 相兼容的，所以也叫 `sysvinit`，这是使用得最悠久的 init 实现。Unix System V 于 1983 年 release。

在 `sysvint` 下，有好几个运行模式，又叫 `runlevel`。比如：常见的 3 级别指定启动到多用户的字符命令行界面，5 级别指定启起到图形界面，0 表示关机，6 表示重启。其配置在 `/etc/inittab` 文件中。

与此配套的还有 `/etc/init.d/` 和 `/etc/rc[X].d`，前者存放各种进程的启停脚本（需要按照规范支持 `start`，`stop`子命令），后者的 X 表示不同的 runlevel 下相应的后台进程服务，如：`/etc/rc3.d` 是 runlevel=3 的。 里面的文件主要是 link 到  `/etc/init.d/` 里的启停脚本。其中也有一定的命名规范：S 或 K 打头的，后面跟一个数字，然后再跟一个自定义的名字，如：`S01rsyslog`，`S02ssh`。S 表示启动，K表示停止，数字表示执行的顺序。
### upstart
co6的服务启动：无依赖关系的服务可并行启动，有依赖关系的顺序启动
### systemd
首先，`systemd` 清醒的认识到了 init 进程的首要目标是要让用户快速的进入可以操作 OS 的环境，所以，这个速度一定要快，越快越好，所以，`systemd` 的设计理念就是两条：
- To start **less**.
- And to start **more** in _parallel_.
也就是说，按需启动，能不启动就不启动，如果要启动，能并行启动就并行启动，包括你们之间有依赖，我也并行启动。按需启动还好理解，那么，有依赖关系的并行启动，它是怎么做到的？这里，`systemd` 借鉴了 MacOS 的 `Launchd` 的玩法

要解决这些依赖性，systemd 需要解决好三种底层依赖—— Socket， D-Bus ，文件系统。
#### Socket 依赖
如果服务C依赖于服务S的 socket，那么就要先启动S，然后再启动C，因为如果C启动时找不到S的 Socket，那么C就会失败。`systemd` 可以帮你在S还没有启动好的时候，建立一个 socket，用来接收所有的C的请求和数据，并缓存之，一旦S全部启动完成，把 systemd 替换好的这个缓存的数据和 Socket 描述符替换过去。
#### D-Bus 依赖
`D-Bus` 全称 Desktop Bus，是一个用来在进程间通信的服务。除了用于用户态进程和内核态进程通信，也用于用户态的进程之前。现在，很多的现在的服务进程都用 `D-Bus` 而不是 Socket 来通信。比如：`NetworkManager` 就是通过 `D-Bus` 和其它服务进程通讯的，也就是说，如果一个进程需要知道网络的状态，那么就必需要通过 `D-Bus` 通信。`D-Bus` 支持 “Bus Activation”的特性。也就是说，A要通过 `D-Bus` 服务和B通讯，但是B没有启动，那么 `D-Bus` 可以把B起来，在B启动的过程中，`D-Bus` 帮你缓存数据。`systemd` 可以帮你利用好这个特性来并行启动 A 和 B。
#### 文件系统依赖
系统启动过程中，文件系统相关的活动是最耗时的，比如挂载文件系统，对文件系统进行磁盘检查（fsck），磁盘配额检查等都是非常耗时的操作。在等待这些工作完成的同时，系统处于空闲状态。那些想使用文件系统的服务似乎必须等待文件系统初始化完成才可以启动。`systemd` 参考了 `autofs` 的设计思路，使得依赖文件系统的服务和文件系统本身初始化两者可以并发工作。`autofs` 可以监测到某个文件系统挂载点真正被访问到的时候才触发挂载操作，这是通过内核 `automounter` 模块的支持而实现的。比如一个 open ()系统调用作用在 “`/misc/cd/file1`” 的时候，`/misc/cd` 尚未执行挂载操作，此时 `open ()` 调用被挂起等待，Linux 内核通知 `autofs`，`autofs` 执行挂载。这时候，控制权返回给 `open ()` 系统调用，并正常打开文件

co7的服务启动：有无依赖关系的服务都并行启动。比co6启动就快，另外，co7/8有时xshell等暂时连接不上，就是因为systemd并行启动服务的原因，有些服务还未真正启动

1. 按需启动守护进程：用户访问就启动，用户不访问不启动
2. pstree -p，可看到是系统的第一个进程
3. 管理系统的所有资源
4. socket与服务程序分离：有些服务有socket（ip:端口号）和service（进程），socket监听不由服务来完成，而由systemd完成，比如rpcbind
![[Pasted image 20230917161808.png]]
![[Pasted image 20230924160655.png]]
- co7中，/sbin/init变成了/lib/systemd/systemd的软链接，ub也用的systemd

- 在co7及以后版本，把各种资源统称为unit
	- service，相当于co6中/etc/init.d下的启动服务脚本
	- target，相当于co6中的 runlevel
![[Pasted image 20230917162817.png]]
![[Pasted image 20230917170709.png]]
### unit的配置文件
- /usr/lib/systemd/system：每个服务最主要的启动脚本设置，该目录下的service unit类似于之前的/etc/init.d/下的启动脚本
![[Pasted image 20230925111024.png]]
所以：/usr/lib/systemd/system = /lib/systemd/system，co7和ub20都是这样
- /run/systemd/system：系统执行过程中所生成的服务脚本，比上面目录优先运行
![[Pasted image 20230925110425.png]]
/run目录并不是真正磁盘上的数据，都是在内存中的，/run/systemd/下的都是运行过程中临时生成的，这个目录一般不去操作

- /etc/systemd/system：管理员建立的执行脚本，类似于/etc/rcN.d/Sxx的功能，比上而日录优先运行。通常是命令自动生成的，不需要手工修改

xinetd：管理非独立服务的代理程序，co6中它负责监听非独立服务的端口号，如果发现非独立服务被访问，就会临时激活对应的非独立服务。值班的。co7有了systemd后就代替了xinetd，xinetd就没用了 

co6中的nfs和rpcbind是有依赖关系的，rpcbind服务必须启动，才能启动nfs
![[Pasted image 20230924162144.png]]
rpcbind如果service rpcbind stop，那么：service nfs start将无法启动，不能自动解决服务之间的依赖关系。而co7是由systemd来统一管理服务，它能自动解决这样的依赖关系，自动激活需要的服务，即直接systemctl start nfs-server，rpcbind也会自动start
## systemctl管理系统服务service unit
```bash
systemctl COMMAND name1.service name2.service
service name COMMAND
而co6中只能同时操作一个服务

禁止自动和手动启动，禁用但不想卸载，手动是无法启动的，如果想再用，可以取消禁用unmask，再启动
systemctl mask name.service

取消禁止
systemctl unmask name.service

查看某服务当前激活与否的状态
systemctl is-active name.service

设定某服务开机自启，相当于chkconfig name on
systemctl enable name.service
查看服务是否开机自启
systemctl is-enabled name.service

设定某服务开机禁止启动：相当于chkconfig name off
systemctl disable name.service

用来列出该服务在哪些运行级别下启用和禁用: chkconfig --list name
Is /etc/systemd/system/*.wants/name.service

列出失败的服务
systemctl --failed --type=service

查看服务的依赖关系，查询到的服务必须先启动，name.service才会启动
systemctl list-dependencies name.service

杀掉进程，用的少，因为有kill
systemctl ki11 unitname

查看所有服务
systemctl list-units --type service --all l-a
```
![[Pasted image 20230925120400.png]]
service; enabled 也表示下次开机自启； vendor preset（出厂预设）: enabled，这个表示刚装好这个软件默认是enable还是disable
### 各种服务状态
```bash
显示所有单元状态
systemctl 或 systemctl list-units
只显示服务单元的状态
systemctl --type=service
显示sshd服务单元
systemctl -1 status sshd.service

重新加载配置到内存中
systemctl reload sshd.service

列出活动状态的所有服务单元
systemctl list-units --type=service

列出所有服务单元systemctl list-units --type=service --all

查看服务单元的启用和禁用状态
systemctl list-unit-files --type=service

查看所有服务的开机自启状态，相当于chkconfig --list
systemctl list-unit-files --type service
显示服务状态，测试上下两条结果貌似一样
systemctl list-unit-files --type service --all
```
 - loaded   Unit配置文件已处理
 - active(running)  一次或多次持续处理的运行；正在运行
 - active(exited)  成功完成一次性的配置，运行了就结束了
 - active(waiting)  运行中，等待一个事件
 - inactive  不运行
 - enabled  开机启动
 - disabled 开机不启动
 - static  开机不启动，但可被另一个启用的服务激活；不能人为/主动启动，只能被别的服务激活/唤醒
 - indirect   重定向到别处
```bash
查看所有已经激活/加载的服务，其中有些可能是failed的
systemctl list-units --type l -t service
```

 ```bash
systemctl enable --now name  =  systemctl enable name;systemctl start name
设为开机自启并立即启动

systemctl disable --now name
设为开机不自启并立即关闭服务

systemctl stop sshd
service sshd stop
co6/7中当下停止了sshd，但已连接的ssh不会马上断开，即已建立的ssh进程不会马上关闭。但新发起的ssh连接会被refused，这是ssh服务自身的特点，其它服务未必如此
```
### service unit文件格式
/etc/systemd/system：系统管理员和用户使用
/usr/lib/systemd/system：发行版打包者使用
帮助参考：systemd.directives (7), systemd.unit(5),systemd.service(5), systemd.socket(5),systemd.target(5),systemd.exec(5)
#### unit 格式说明：
- 以"#"开头的行后面的内容会被认为是注释
- 相关布尔值，1、yes、on、true 都是开启，0、no、off、false 都是关闭
- 时间单位默认是秒，所以要用毫秒(ms)分钟(m)等需显式说明
#### service unit file文件通常由三部分组成：
- [Unit]：定义与Unit类型无关的通用选项;用于提供unit的描述信息、unit行为及依赖关系等
- [Service]：与特定类型相关的专用选项；此处为Service类型
- [Install]：定义由"systemcti enable"以及"systemctl disable"命令在实现服务启用或禁用时用到的一些选项
#### Service段的常用选项：
- Type：定义影响ExecStart及相关参数的功能的unit进程启动类型
	- simple：默认值，这个daemon主要由ExecStart接的指令串来启动，启动后常驻于内存中
	- forking：由ExecStart启动的程序透过spawns延伸出其他子程序来作为此daemon的主要服务。原生父程序在启动结束后就会终止。**由forking对应的进程衍生出一个子进程，而不是由进程自身来提供服务，原生的父进程就停止了
	- oneshot：与simple类似，不过这个程序在工作完毕后就结束了，不会常驻在内存中。一次性的
	- dbus：与simple类似，但这个daemon必须要在取得一个D-Bus的名称后，才会继续运作.因此通常也要同时设定BusNname= 才行
	- notify：在启动完成后会发送一个通知消息。还需要配合 NotifyAccess 来让 Systemd 接收消息
	- idle：空闲。与simple类似，要执行这个daemon必须要所有的工作都顺利执行完毕后才会执行。这类的daemon通常是开机到最后才执行即可的服务
- EnvironmentFile：环境配置文件。通常定义了变量
- ExecStart：指明启动unit要运行命令或脚本的**绝对路径
- ExecStartPre: ExecStart前运行
- ExecStartPost: ExecStart后运行
- ExecStop:指明停止unit要运行的命令或脚本
- Restart：当设定Restart=1 时，则当次daemon服务意外终止后，会再次自动启动此服务
- PrivateTmp：设定为yes时，会生成/tmp/systemd-private-UUID-NAME.service-XXXXX/tmp/目录。 用来存放临时文件
#### Install段的常用选项：
- Alias：别名，可使用systemctl command Alias.service
- RequiredBy：被哪些units所依赖，强依赖
- WantedBy：被哪些units所依赖，弱依赖
- Also：安装本服务的时候还要安装别的相关服务
- 注意：对于新创建的unit文件，或者修改了的unit文件，要通知systemd重载此配置文件：systemctl daemon-reload，而后可以选择重启
#### Unit段的常用选项：
- Description：描述信息
- After：定义unit的启动次序，表示当前unit应该晚于哪些unit启动，即：依赖于哪些unit，其功能与Before相反
- Requires：依赖到的其它units，强依赖，被依赖的units无法激活时，当前unit也无法激活
- Wants：依赖到的其它units，弱依赖
- Conflicts：定义units间的冲突关系。本服务启动就不能启动其它服务，比如：sendmail（较老）和postfix
#### 例：/usr/lib/systemd/system/atd.service
- After=syslog.target systemd-user-sessions.service
变相说明了依赖关系，表示atd是在syslog.target systemd-user-sessions.service这些服务后运行的

- [Service]
如果是socket资源，就写socket
- ExecStart=/usr/sbin/atd -f $OPTS
$OPTS 这是调用的环境变量，从EnvironmentFile=/etc/sysconfig/atd调用，该文件中#OPTS="-l 4 -b 120"被注释了，就是空值

- IgnoreSIGPIPE=no
忽略信号，这个和服务有关，服务是否支持信号

- [Install]
如果没有 [Install]，表示仅是一个文件，不起作用；[Install]了，才生效

- WantedBy=multi-user.target
被multi-user.target（level 3）所依赖，WantedBy是个弱依赖，即：multi-user.target要启动，要启动atd.service。**开机进入到level 3，希望启动此服务**
#### 例：nginx.service
![[Pasted image 20230925145422.png]]
![[Pasted image 20230925145445.png]]
#### 例：tomcat.service
![[Pasted image 20230925145545.png]]
#### 例：bak.service
![[Pasted image 20230925150120.png]]
at now：作一次备份
#### 不写service文件实现服务开机自启：/etc/rc.local
co7可用
ub20默认没有此文件，但可手工创建并加执行权限，也能用
```bash
[root@ubuntu1804 ~] cat /etc/rc.local
#!/bin/bash
echo -e '\E[31;1mstarting test service\E[Om'
sleep 10
[root@ubuntu1804 ~] chmod +x /etc/rc.local
```
### 运行级别
target units：相当于Centos 6之前的runlevel ,unit配置文件: .target
```bash
ls /usr/lib/systemd/system/*.target
systemctl list-unit-files --type target --all
```
**和运行级别对应关系**
0==> runlevel0.target, poweroff.target
1 ==> runlevel1.target, rescue.target
2 ==> runlevel2.target, multi-user.target
3 ==> runlevel3.target, multi-user.target
4 ==> runlevel4.target, multi-user.target
5 ==> runlevel5.target, graphical.target
6 ==> runlevel6.target, reboot.target
\#  每个target就代表着某些资源的集合，某个服务在某种target是否启动，是可以单独指定的
## systemd启动流程和grub2管理及启动故障排错
systemctl list-dependencies graphical.target
\#  列出某个target涵盖的资源
![[Pasted image 20230917190536.png]]
systemctl default
\#  回到开机进入的模式

**co7及以后版本，init 2,3,4都一样，互为软链接了**
![[Pasted image 20230917191410.png]]
rescue相当于1模式
![[Pasted image 20230926140259.png]]
![[Pasted image 20230917191722.png]]
![[Pasted image 20230917191809.png]]
mask：禁用；init q ：使之生效（7/8上使用可能会出问题），相当于 systemctl daemon-reload
## co7及其后版本引导顺序
![[Pasted image 20230917192638.png]]
7、~~执行initrd.target，相当于co6的/etc/rc.d/rc.sysinit~~
![[Pasted image 20230917192812.png]]
systemd-analyze plot > boot.xml
\#  生成一个xml格式的文件
sz boot.xml   传到win后，用浏览器打开 
![[Pasted image 20230917205717.png]]

systemd-analyze blame
\#  字符界面，可以看到每个服务花费了多少时间

![[Pasted image 20230917210121.png]]
如某个服务出故障了，导致系统无法正常启动，类似co6，启动菜单按e，加a，输个数字如1，这样本次临时性的进入1/rescue模式，1模式那个服务是不启动的，这样就绕过了故障，然后禁用这个故障服务 
### 破解co7/8的root密码
![[Pasted image 20230917210644.png]]
rd.break：临时打断当前linux正常启动，不进入到正常启动状态，进入特殊模式，在这个模式下，不需要输入口令就可进入
现在/sysroot才是真正硬盘的根，chroot /sysroot后，也可直接vi /etc/shadow改密码
![[Pasted image 20230917211412.png]]
### 给grub2菜单加口令
![[Pasted image 20230917211505.png]]
会生成一个user.cfg文件，去除密码，直接删除这个文件也行
设了以后就不能按e修改root密码了
![[Pasted image 20230917211655.png]]
### 修复grub2
![[Pasted image 20230917212239.png]]
![[Pasted image 20230917212631.png]]
grub2-install 难以实现
### 调整默认启动内核
![[Pasted image 20230926144806.png]]
grubenv   定义了默认的启动内核
![[Pasted image 20230917214443.png]]
![[Pasted image 20230917214240.png]]
 co8或直接手工修改：/boot/grub2/grubenv，替换saved_entry=后的字符即可。co7没有 /boot/loader/entries目录，
### 例：co7/8破坏mbr后恢复
![[Pasted image 20230917212757.png]]
### 例：co7/8删除/boot/grub2/\*所有内容进行恢复
![[Pasted image 20230917214609.png]]
grub2-mkconfig借助一个模板文件：/etc/default/grub自动生成grub.cfg
### 例：co7/8删除/boot/下所有文件后恢复
![[Pasted image 20230917214802.png]]
![[Pasted image 20230917214820.png]]
![[Pasted image 20230917215101.png]]
或：grub2-mkconfig > /boot/grub2/grub.cfg   #  用重定向也行
\#  单独执行grub2-mkconfig就是输出菜单文件的内容
