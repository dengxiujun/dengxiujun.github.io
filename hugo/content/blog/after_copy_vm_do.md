+++ 
date = 2019-04-01T10:10:40+08:00
title = "VMware虚拟机转VitrualBOX虚拟机等后续优化操作"
description = "VMware虚拟机转VitrualBOX虚拟机等后续优化操作，包括扩硬盘"
slug = "" 
tags = ["vm", "vbox", "virtual box"]
categories = []
externalLink = ""
series = []
+++


### 01. VMware虚拟机转VitrualBox虚拟机

01) 打开cmd，并切换到VMware的安装目录下的OVFTool目录
```
d: # 盘符切换
cd D:\VMware\VMware Workstation\OVFTool
```
02) 使用 ovftool 命令转换格式
```
ovftool "D:\Virtual Machines\prd_170_vm\prd_170.vmx" "D:\VirtualMachines\prd_170_vb\prd_170.ovf"
```
注：需要 D:\VirtualMachines\prd_170_vm\prd_170.vmx 该虚拟机处于关机状态

03) 打开VitrualBOX主界面，依次打开"管理">>"导入虚拟电脑"，在弹出界面中选择 D:\VirtualMachines\prd_170_vb\prd_170.ovf

04) 处理网卡配置等，如果有安装"VM增强插件"，可以删除之后安装"VitrualBox增强插件"

### 02. VMware磁盘格式vmdk转VirtualBox磁盘格式vdi

01) 打开cmd，并切换到VitrualBox安装目录
```
d: # 盘符切换
cd D:\VirtualBox
```
02) 使用 VBoxManage.exe clonehd 命令克隆并格式化磁盘格式
```
VBoxManage.exe clonehd "D:\Virtual Machines\prd_6.170_vb\prd_6.170.vmdk" "D:\Virtual Machines\prd_6.170_vb\prd_6.170.vdi" --format VDI
```
03) 上述命令执行到90%的时间点会报错误，错误信息如下，原因是克隆的磁盘和原磁盘uuid重复，此时可以修改原磁盘和新磁盘其中一个的uuid，使用命令如下
```
Progress state: E_INVALIDARG
VBoxManage.exe: error: Failed to clone medium
VBoxManage.exe: error: Cannot register the hard disk 'D:\Virtual Machines\prd_6.170_vb\prd_6.170.vdi' {4cb06e65-d214-4361-b9c6-b5faf1f5effa} because a hard disk 'D:\Virtual Machines\prd_6.170_vb\prd_6.170.vdi' with UUID {0160913f-6e83-4cd6-97b7-c9e24d9cd26d} already exists
VBoxManage.exe: error: Details: code E_INVALIDARG (0x80070057), component VirtualBoxWrap, interface IVirtualBox
VBoxManage.exe: error: Context: "enum RTEXITCODE __cdecl handleCloneMedium(struct HandlerArg *)" at line 954 of file VBoxManageDisk.cpp
VBoxManage.exe internalcommands sethduuid "D:\Virtual Machines\prd_6.170_vb\prd_6.170.vdi"
```
04) 打开VirtualBox主界面，依次选择"管理">>"虚拟介质管理"，在弹出界面中选中原来的vmdk磁盘并使用右键释放和删除该vmdk介质
![102301_K5s3_2884716](/images/after_copy_vm_do/102301_K5s3_2884716.png "")

05) 打开转换后的虚拟机的设置界面，"存储">>"控制器">>"添加虚拟硬盘"，然后在弹出界面中选择转换后的dvi磁盘
![103101_E4VE_2884716](/images/after_copy_vm_do/103101_E4VE_2884716.png "")

### 03. 磁盘扩容
方案一：使用"VBoxManage modifyhd"命令

打开cmd，并切换到VitrualBox安装目录执行以下命令

VBoxManage modifyhd "D:\Virtual Machines\prd_6.170\prd_6.170.vdi" –-resize 102400

注：该命令只支持vdi格式的磁盘

方案二：针对不同操作系统采用新增磁盘挂载方式

linux参考 http://www.linuxidc.com/Linux/2011-02/32083.htm

windows请自行百度

