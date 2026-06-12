---
title: Windows Server 测试系统常用设置
date: 2026-05-18 15:00:05
description: Windows Server 测试系统常用设置
tags:
  - Windows Server
categories:
  - Uncategorized
---
## 禁用 IPv6

Win + R 打开注册表 regedit

![regedit](Windows-Server-System-Common-Settings/regedit.png)

访问：计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters

![新建](Windows-Server-System-Common-Settings/新建.png)

名称为：DisabledComponents

![DisabledComponents](Windows-Server-System-Common-Settings/DisabledComponents.png)

双击修改，设置值为：ffffffff

![设置值ffffffff](Windows-Server-System-Common-Settings/设置值ffffffff.png)

重启后生效

## 限制 DEP

![高级系统设置](Windows-Server-System-Common-Settings/高级系统设置.png)

![性能设置](Windows-Server-System-Common-Settings/性能设置.png)

![限制DEP](Windows-Server-System-Common-Settings/限制DEP.png)

## 禁用防火墙

![打开防火墙](Windows-Server-System-Common-Settings/打开防火墙.png)

![防火墙属性](Windows-Server-System-Common-Settings/防火墙属性.png)

将域配置文件、专用配置文件、公用配置文件的防火墙状态都设置为关闭

![关闭状态](Windows-Server-System-Common-Settings/关闭状态.png)

## 禁用实时保护

![打开防病毒页面](Windows-Server-System-Common-Settings/打开防病毒页面.png)

![禁用实时保护](Windows-Server-System-Common-Settings/禁用实时保护.png)

## 禁用IE的增强安全设置

![打开IE安全设置](Windows-Server-System-Common-Settings/打开IE安全设置.png)

![禁用IE安全设置](Windows-Server-System-Common-Settings/禁用IE安全设置.png)

## 更改用户账户控制设置

![更改用户账户控制设置](Windows-Server-System-Common-Settings/更改用户账户控制设置.png)

![从不通知](Windows-Server-System-Common-Settings/从不通知.png)

## 电源选项

![电源选项](Windows-Server-System-Common-Settings/电源选项.png)

## 关闭 cmd 的快速编辑模式

打开 cmd，邮件窗口，打开属性

![cmd属性](Windows-Server-System-Common-Settings/cmd属性.png)

![关闭快速编辑模式](Windows-Server-System-Common-Settings/关闭快速编辑模式.png)
