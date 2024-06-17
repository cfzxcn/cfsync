方案一：修改bios启动项后重启，更换主板电池

**Dell安装Ubuntu系统的坑**

- 采用UEFI方式安装，dell一般只支持这一种方式安装。
- 修改bios设置中的硬盘方式，将raid on 方式改为ahci方式。不然会报内存空间不足的问题，不会将系统安装到硬盘上。
- 安装ubuntu的时候，分区时一定要注意选择划分一个efi分区，内存大概在200M左右既可，不然会报grub错误。
- 安装好系统之后，重新启动机器，会报no boot 错误。解决方式为开机按f12，进入bios设置，选择boot sequence，选择UEFI，然后选择add boot system，然后命名为Ubuntu，文件选择EFI文件目录下的grub.的那个文件。
- 重新启动，即可进入安装好的Ubuntu系统。

**方案二：**

登录到 iDRAC 以查看日志。此外，查看生命周期日志，因为这些日志也可为故障处理提供帮助。  
**iDRAC9**：转至 Maintenance，然后转至 **LifeCycle Logs + System Event Logs**。  
**iDRAC8**：转至 **Logs + LifeCycle Logs**