---
title: 升级 BMIDE 项目
date: 2025-09-15 10:35:49
description: 升级 BMIDE 项目
tags:
  - BMIDE
  - 升级
categories: 
  - PLM
  - BMIDE
---

## 1 为什么要升级

DC 会检查 BMIDE 项目的依赖关系，比如之前部署的是依赖于 TC2412.0007 版本的模板，TC 升级后小版本就变为了 0008，再次部署 BMIDE 时由于找不到 0007 的依赖版本，可能会导致在应用程序选项卡中勾选不了我们的模板项目。

![版本不一致](Upgrade-BMIDE-Project/版本不一致.png)

## 2 升级步骤

![重新运行模板项目升级向导](Upgrade-BMIDE-Project/重新运行模板项目升级向导.png)

![模板项目升级](Upgrade-BMIDE-Project/模板项目升级.png)

![升级并加载项目](Upgrade-BMIDE-Project/升级并加载项目.png)

![升级成功](Upgrade-BMIDE-Project/升级成功.png)

## 3 验证

重新生成部署软件包后，可以看到小版本号发生变化：

![验证](Upgrade-BMIDE-Project/验证.png)
