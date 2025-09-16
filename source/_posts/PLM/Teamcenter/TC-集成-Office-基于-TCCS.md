---
title: TC 集成 Office 基于 TCCS
date: 2025-09-15 14:33:13
tags:
  - Teamcenter
  - 集成
---

## 1 添加组件

在 DC 的组件选项卡中，添加 Teamcenter Client for Office

![Office组件1](./TC-集成-Office-基于-TCCS/Office组件1.png)

![Office组件2](./TC-集成-Office-基于-TCCS/Office组件2.png)

如果部署的客户端已经有 TCCS 了，那么不需要再次配置 TCCS，只需确认开启了客户端通信配置即可。如果没有，则需要配置：

![TCCS组件1](./TC-集成-Office-基于-TCCS/TCCS组件1.png)

![TCCS组件2](./TC-集成-Office-基于-TCCS/TCCS组件2.png)

## 2 部署

正常生成部署脚本部署即可
