# style setting 分屏
ctrl+p，输入：style，拖动标题栏向左侧

https://www.cnblogs.com/yantul/p/18127305
# 代码块
代码第1个字符前按4个空格，感觉不好用

    subnet 192.168.1.0 netmask 255.255.255.0 {
	range 192.168.1.200   192.168.1.219; 
	range 192.168.1.231   192.168.1.253;
	option routers 192.168.1.2;   # 网关
	}
# 代码框
```bash
sed -i '14s#yes#no#'/etc/xinetd.d/tftp
systemctl enable tftp.service --now
systemctl status tftp.service
```
14s#yes#no#：Code block string color：#BB1111
--now：Code block attribute color：
sed：Code block builtin color：
# 字体-font设置不生效
## 字体
`<font face="KAI">楷体</font>`
`黑体<font face='SimHei'>
`宋体<font face='SimSun'>
`新宋体<font face='NSimSun'>
`仿宋<font face='FangSong'>
`楷体<font face='KaiTi'>
`仿宋_GB2312<font face='FangSong_GB2312'>
`楷体_GB2312<font face='KaiTi_GB2312'>
`微软雅黑<fontface='MicrosoftYaHei'\>
## 字号
<big>三种</big>方式修改字号：
`<font size=6>第一种</font>是使用<font>标签`
`第二种通过<big>或者<small>标签`
第三种是通过修改style样式实现
## 颜色
<font color=green>style</font>
<font style="background: yellow " color=green>背景</font>
# 图片
![[Pasted image 20240424011853.png]]
# 引用

>你好
# 添加文档属性
第一行添加三个横线






