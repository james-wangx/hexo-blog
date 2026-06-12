---
title: 安装 MS SQL Server 2022
date: 2026-06-12 07:47:56
description: 安装 MS SQL Server 2022
tags:
  - 安装
  - MS SQL Server
categories:
  - 未分类
---

## 1 安装包下载

浏览器打开：https://www.microsoft.com/zh-cn/download/details.aspx?id=104781

![image-20260612092036729](./安装-MS-SQL-Server-2022/下载地址.png)

## 2 自定义（有网络安装）

指定一个媒体的位置

![image-20260612082818932](./安装-MS-SQL-Server-2022/自定义.png)

## 3 下载介质（无网络安装）

打开下载好的 SQL2022-SSEI-Expr.exe，如果当前机器没有网络，则需要在有网络的机器上下载介质。如果有网络，则直接点击自定义。

**如果选择自定义，可以跳过这一节。**

![image-20260612082021946](./安装-MS-SQL-Server-2022/下载介质.png)

如果点击下载介质，选择 Express Advanced

![image-20260612082114018](./安装-MS-SQL-Server-2022/Express Advanced.png)

下载完成后，将介质放到要安装的机器上并打开。提取到指定目录：

![image-20260612082149305](./安装-MS-SQL-Server-2022/提取文件.png)

## 4 安装步骤

自定义或者使用介质的方式都会自动弹出该安装程序

![image-20260612083155340](./安装-MS-SQL-Server-2022/开始安装.png)

![image-20260612083504094](./安装-MS-SQL-Server-2022/接受许可条款.png)

如果没有网络，可以不选择检查更新

![image-20260612083558876](./安装-MS-SQL-Server-2022/检查更新.png)

没有网络就跳过

![image-20260612084055745](./安装-MS-SQL-Server-2022/没有网络跳过.png)

![image-20260612084138258](./安装-MS-SQL-Server-2022/功能选择.png)

选择默认实例

![image-20260612084217205](./安装-MS-SQL-Server-2022/实例配置.png)

勾选授予“执行卷维护任务”特权

![image-20260612092514812](./安装-MS-SQL-Server-2022/服务器配置.png)

选择混合模式并设置 sa 账户密码

![image-20260612084339092](./安装-MS-SQL-Server-2022/数据库引擎配置.png)

安装成功：

![image-20260612084602747](./安装-MS-SQL-Server-2022/安装成功.png)

## 5 安装 SSMS

### 5.1 下载 SSMS

浏览器打开：https://learn.microsoft.com/zh-cn/ssms/install/install

![image-20260612093207379](./安装-MS-SQL-Server-2022/SMMS下载地址.png)

### 5.2 有网络安装

![SSMS安装位置](./安装-MS-SQL-Server-2022/SSMS安装位置.png)

### 5.3 无网络安装

如果没有网络，则需要先到有网络的机器上创建脱机安装缓存，cmd 运行（layout 替换为合适的路径）：

```cmd
vs_SSMS.exe --layout C:\Users\james\Downloads\SSMS_Layout --add Microsoft.Component.HelpViewer --locale zh-CN
```

下载完成

![image-20260612094315650](./安装-MS-SQL-Server-2022/Layout下载完成.png)

将 SSMS_Layout 打包放到目标机器上

下载证书：https://learn.microsoft.com/zh-cn/ssms/install/install-certificates#validate-a-certificate-for-offline-installations

![image-20260612094434932](./安装-MS-SQL-Server-2022/下载证书.png)

在目标机器上安装证书

![image-20260612094601551](./安装-MS-SQL-Server-2022/安装证书.png)

![image-20260612094617586](./安装-MS-SQL-Server-2022/选择本地计算机.png)

![image-20260612094635580](./安装-MS-SQL-Server-2022/导入证书.png)

![image-20260612094653123](./安装-MS-SQL-Server-2022/导入成功.png)

目标机器上打开 cmd 到 SSMS_Layout 目录下运行：

```cmd
vs_SSMS.exe --noWeb --add Microsoft.Component.HelpViewer
```

弹出 Visual Studio Installer，选择安装位置

![SSMS安装位置](./安装-MS-SQL-Server-2022/SSMS安装位置.png)

安装完毕后重启

![SSMS安装完毕](./安装-MS-SQL-Server-2022/SSMS安装完毕.png)

### 5.4 验证

可以选择不登录

![验证连接](./安装-MS-SQL-Server-2022/验证连接.png)

#### 5.4.1 Windows 身份验证

服务器名称为当前主机名，勾选信任服务器证书

![Windows身份验证](./安装-MS-SQL-Server-2022/Windows身份验证.png)

![image-20260612100410423](./安装-MS-SQL-Server-2022/Windows身份验证成功.png)

#### 5.4.2 SQL Server 身份验证

![新建连接](./安装-MS-SQL-Server-2022/新建连接.png)

选择 SQL Server 身份验证，输入 sa 的密码

![SQL Server 身份验证](./安装-MS-SQL-Server-2022/SQL Server 身份验证.png)

![SQL Server 身份验证成功](./安装-MS-SQL-Server-2022/SQL Server 身份验证成功.png)

## 6 开启 TCP/IP

打开 “SQL Server 2022 配置管理器“，开启 TCP/IP，默认端口是 1433

![配置管理器](./安装-MS-SQL-Server-2022/配置管理器.png)

![用 TCP IP](./安装-MS-SQL-Server-2022/启用 TCP IP.png)

![重启服务](./安装-MS-SQL-Server-2022/重启服务.png)

重启服务，尝试从其它机器上连接：

![重启 SQL Server 服务](./安装-MS-SQL-Server-2022/重启 SQL Server 服务.png)

例如使用 DateGrip 连接，输入 host、port、用户名和密码后测试连接成功

![测试连接成功](./安装-MS-SQL-Server-2022/测试连接成功.png)
