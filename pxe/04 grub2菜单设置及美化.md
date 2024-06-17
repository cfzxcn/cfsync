## GRUB2变量
- default：默认选中第几个菜单项(从'0'开始计数)
- fallback：如果默认菜单项启动失败，那么就启动第几个菜单项(从'0'开始计数)
- timeout：在启动默认菜单项前，等待键盘输入的秒数。默认值是'5'秒。'0'表示直接启动默认菜单项(不显示菜单)，'-1'表示永远等待。
- gfxmode：设置"gfxterm"模块所使用的视频模式，可以指定一组由逗号或分号分隔的模式以供逐一尝试：每个模式的格式必须是：'auto'(自动检测),'宽x高','宽x高x色深'之一，并且只能使用VBE标准指定的模式[640x480,800x600,1024x768,1280x1024]x[16,24,32]。可以在GRUB SHELL中使用"videoinfo"命令列出当前所有可用模式。默认值是'auto'。例：set gfxmode=1024x768,auto
- gfxpayload：设置Linux内核启动时的视频模式，可以指定一组由逗号或分号分隔的模式以供逐一尝试：每个模式的格式必须是：'text'(普通文本模式,不能用于UEFI平台),'keep'(继承"gfxmode"的值),'auto'(自动检测),'宽x高','宽x高x色深'之一，并且只能使用VBE标准指定的模式[640x480,800x600,1024x768,1280x1024]x[16,24,32]。在BIOS平台上的默认值是'text'，在UEFI平台上的默认值是'auto'。
- 设置自定义变量不需要括起来，如：set IP=192.168.1.3，但使用变量必须用双引号，无引号及单引号均出问题，如：url="h
ttp://${IP}/ks/db913-preseed.cfg"
## GRUB2模块
insmod 命令来载入模块
视频模块：提供了各种不同的视频模式支持，一共6个。例如：vga vbe efi_gop efi_uga ...
终端模块：serial   gfxterm   vga_text   at_keyboard ...
命令模块：提供了各种不同的功能，类似标准Unix命令，一共将近100个。例如：cat cpuid echo halt lspci chainloader initrd linux password ... 。在普通模式中，命令模块[command.lst]与加密模块[crypto.lst]会被自动按需载入(无需使用"insmod"命令)，并且可使用完整的GRUB脚本功能。但是其他模块则可能需要明确使用"insmod"命令来载入
## GRUB2命令
terminal_output gfxterm
指定了gfxterm终端，表示将gfxterm终端设置为当前激活的输出终端
`menuentry "title" [--class=class …] [--users=users] [--unrestricted] [--hotkey=key] [--id=id] [arg …] { command; … }`
定义一个名为"title"的菜单项。当此菜单项被选中时，GRUB将会执行花括号中的命令列表
可以使用 --class 选项指定菜单项所属的"样式类"。从而可以使用指定的主题样式显示菜单项。
可以使用 --users 选项指定只允许特定的用户访问此菜单项。如果没有使用此选项，则表示允许所有用户访问。
可以使用 --unrestricted 选项指明允许所有用户访问此菜单项。
可以使用 --hotkey 选项设置访问此菜单项的热键(快捷键)。"key"可以是一个单独的字母，或者'backspace','tab','delete'之一。
可以使用 --id 选项为此菜单项设置一个全局唯一的标识符。"id"必须由ASCII字母/数字/下划线组成，且不得以数字开头。
[arg …]是可选的参数列表。你可以把它们理解为命令行参数。实际上"title"也是命令行参数，只不过这个参数是个必须参数而已。这些参数都可以在花括号内的命令列表中使用，"title"对应着"$1"，其余的以此类推。
linux file …
使用32位启动协议从"file"载入一个Linux内核映像，并将其余的字符作为内核的命令行参数逐字传入。
[注意]使用32位启动协议意味着'vga='启动选项将会失效。如果你希望明确设置一个特定的视频模式，那么应该使用"gfxpayload"环境变量。虽然GRUB可以自动地检测某些'vga='参数，并把它们翻译为合适的"gfxpayload"设置，但是并不建议这样做。
initrd file
为以32位协议启动的Linux内核载入一个"initial ramdisk"，并在内存里的Linux设置区域设置合适的参数。[注意]这个命令必须放在"linux"命令之后使用。
loadfont file …
从指定的"file"加载字体
set [envvar=value]
将环境变量"envvar"的值设为'value'。如果没有使用参数，则打印出所有环境变量及其值。
## 更改字体及大小
https://www.linuxmi.com/linux-change-grub-menu-font-size.html
不能简单地将 TTF、OTF、WOFF 等格式的字体直接用于 GRUB。相反，它使用扩展名为 PF2 的特定格式字体。它专门设计用于标准操作系统服务和驱动程序不可用的引导前环境。

下载字体：
wget http://sourceforge.net/projects/dejavu/files/dejavu/2.37/dejavu-fonts-ttf-2.37.zip

unzip dejavu-fonts-ttf-2.37.zip
root@ub20:~/grub-font# ls
dejavu-fonts-ttf-2.37  dejavu-fonts-ttf-2.37.zip
root@ub20:~/grub-font# 
root@ub20:~/grub-font# cd dejavu-fonts-ttf-2.37/
root@ub20:~/grub-font/dejavu-fonts-ttf-2.37# ls
AUTHORS  BUGS  LICENSE  NEWS  README.md  fontconfig  langcover.txt  status.txt  ttf  unicover.txt

运行以下命令将字体转换为 PF2 格式：
root@ub20:~/grub-font/dejavu-fonts-ttf-2.37# grub-mkfont -s 22 -o dejavu-sans-mono.pf2 ttf/DejaVuSansMono.ttf
 -s：设置生成字体的大小
 -o：输出为一个文件，可指定路径
 最后一个参数是TTF 文件的完整路径
 
root@ub20:~/grub-font/dejavu-fonts-ttf-2.37# scp dejavu-sans-mono.pf2 192.168.1.79:/var/lib/tftpboot/grub

/var/lib/tftpboot/grub/grub.cfg中的设置：
```
if loadfont /grub/dejavu-sans-mono.pf2;then
#if loadfont /grub/font.pf2;then
        set gfxmode=auto
#       set gfxmode=1600x1050
#       set gfxmode=1440x900
        insmod efi_gop
        insmod efi_uga
        insmod gfxterm
        terminal_output gfxterm

        insmod video_bochs
        insmod video_cirrus
        insmod png
fi
```
## 更改颜色
https://bbs.huaweicloud.com/blogs/364516
可以更改的 3 种主要 GRUB 颜色设置。
```
set menu_color_normal=white/cyan
# 未选中菜单项的颜色,菜单框背景颜色
set menu_color_highlight=blue/white
#set menu_color_highlight=yellow/blue
# 突出显示的菜单项的颜色及其在菜单框中的背景
set color_normal=yellow/black
# 菜单框外文字和背景的颜色
```
grub 支持以下颜色：
```javascript
black
blue
brown
cyan
dark-gray
green
light-cyan
light-blue
light-green
light-gray
light-magenta
light-red
magenta
red
white
yellow
```
不要更改 color_normal 中的“黑色”。如果更改，则显示菜单的区域中的图像将不透明
## 各种测试
1、不能写成：inst.ks="/kernel/co7/co7.graphical.lvm.cfg"，否则提示如下：
![[Pasted image 20240509051449.png]]
## 不能使用的快捷键
c

![[Pasted image 20240407040132.png]]