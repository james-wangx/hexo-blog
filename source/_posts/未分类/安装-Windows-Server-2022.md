---
title: 安装 Windows Server 2022
date: 2025-10-03 15:51:11
tags:
  - 安装
  - Windows Server
category: 未分类
---

## 1 准备镜像文件

进入 MSDN，找到 Windows Server 2022，选择 BT 种子下载

https://next.itellyou.cn/Original/#cbp=Product?ID=ff70d59a-8e02-ec11-a9e5-95b21d9a899a

![MSDN](./安装-Windows-Server-2022/MSDN.png)

magnet:?xt=urn:btih:40810feeda21cb5fbcfa2f4eebf2fd9356378412&dn=zh-cn_windows_server_2022_updated_sep_2024_x64_dvd_cab4e960.iso&xl=6155229184

使用 qBittorrent 等工具下载

## 2 新建虚拟机

如果是物理机安装，可以跳过这一步。

打开 VMware，Ctrl + N，或者点击右上角 文件 -> 新建虚拟机

![新建虚拟机](./安装-Windows-Server-2022/新建虚拟机.png)

选择自定义(高级)，模拟真实的物理机安装环境

![自定义高级](./安装-Windows-Server-2022/自定义高级.png)

![硬件兼容性](./安装-Windows-Server-2022/硬件兼容性.png)

选择稍后安装操作系统，否则会立刻开始激活和创建用户

![稍后安装操作系统](./安装-Windows-Server-2022/稍后安装操作系统.png)

![操作系统版本](./安装-Windows-Server-2022/操作系统版本.png)

虚拟机名称和位置请自定义

![命名虚拟机](./安装-Windows-Server-2022/命名虚拟机.png)

![固件类型](./安装-Windows-Server-2022/固件类型.png)

处理器配置和内存配置根据宿主机的配置和自身需要自定义

![处理器配置](./安装-Windows-Server-2022/处理器配置.png)

![内存配置](./安装-Windows-Server-2022/内存配置.png)

![网络类型](./安装-Windows-Server-2022/网络类型.png)

![IO控制器类型](./安装-Windows-Server-2022/IO控制器类型.png)

![磁盘类型](./安装-Windows-Server-2022/磁盘类型.png)

![选择磁盘](./安装-Windows-Server-2022/选择磁盘.png)

磁盘大小根据自身需要自定义

![磁盘大小](./安装-Windows-Server-2022/磁盘大小.png)

![磁盘文件](./安装-Windows-Server-2022/磁盘文件.png)

![准备创建](./安装-Windows-Server-2022/准备创建.png)

虚拟机的处理器、内存、磁盘的使用量是由当前虚拟机使用状态决定的，配置中设置的只是上限，不会立刻占满所有空间。

右键创建好的虚拟机打开设置，选择合适的镜像文件

![镜像文件](./安装-Windows-Server-2022/镜像文件.png)

## 3 开始安装

开启机器，按下任意按键，从镜像文件启动

![从镜像文件启动](./安装-Windows-Server-2022/从镜像文件启动.png)

![语言设置](./安装-Windows-Server-2022/语言设置.png)

![现在安装](./安装-Windows-Server-2022/现在安装.png)

选择我没有产品密钥

![没有产品密钥](./安装-Windows-Server-2022/没有产品密钥.png)

![Datacenter](./安装-Windows-Server-2022/Datacenter.png)

![许可条款](./安装-Windows-Server-2022/许可条款.png)

选择自定义安装

![自定义安装](./安装-Windows-Server-2022/自定义安装.png)

![安装位置](./安装-Windows-Server-2022/安装位置.png)

等待安装完成后会自动重启

![正在安装](./安装-Windows-Server-2022/正在安装.png)

![立即重启](./安装-Windows-Server-2022/立即重启.png)

重启两次会设置管理员账户密码

![设置密码](./安装-Windows-Server-2022/设置密码.png)

![安装完成](./安装-Windows-Server-2022/安装完成.png)

