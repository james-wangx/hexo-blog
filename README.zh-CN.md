# James Wang 的博客

![Hexo](https://img.shields.io/badge/Hexo-8.1.2-0e83cd)
![NexT](https://img.shields.io/badge/NexT-8.28.0-0e83cd)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Active-222)
![License](https://img.shields.io/badge/License-MIT-blue)

基于 Hexo 8 + NexT 8 的个人技术博客，使用 GitHub Pages 部署。

在线地址：<https://james-wangx.github.io>

[English](README.md)

## 技术栈

- Hexo 8 静态站点生成器
- NexT 8 主题
- GitHub Pages 部署
- hexo-generator-searchdb 站内搜索
- hexo-generator-sitemap 站点地图
- hexo-next-giscus 评论
- hexo-relative-link 相对路径图片链接

## 本地开发

安装依赖：

```bash
npm install
```

常用命令：

| 命令 | 说明 |
| --- | --- |
| `npm run server` | 本地预览，默认 http://localhost:4000 |
| `npm run build` | 生成静态文件到 `public` |
| `npm run clean` | 清理生成缓存（`public`、`db.json`） |
| `npm run deploy` | 部署 `public` 到 GitHub Pages |

完整部署流程：

```bash
npm run clean
npm run build
npm run deploy
```

## 写文章

文章放在 `source/_posts/` 下，按分类建立子目录，例如：

```text
source/_posts/PLM/Teamcenter/My-Article.md
```

写作规范：

- 标题使用英文，文件名用 `-` 代替空格。
- Front matter 中通过 `categories` 指定分类，例如 `PLM` + `Teamcenter`。
- 图片粘贴到与文章同名的目录中，Markdown 中使用相对路径引用：

```markdown
![alt](./My-Article/image.png)
```

Front matter 示例：

```yaml
---
title: Create Resource Assignments to Tasks in the Schedule Using ITK
description: Create resource assignments to tasks in the schedule using ITK
date: 2026-08-04 10:22:05
tags:
  - Teamcenter
  - ITK
  - Schedule
categories:
  - PLM
  - Teamcenter
---
```

## 部署

部署目标在 `_config.yml` 中配置：

```yaml
deploy:
  type: git
  repo: https://github.com/james-wangx/james-wangx.github.io
  branch: master
```

执行 `npm run deploy` 后，`public` 目录中的内容会推送到 `james-wangx.github.io` 仓库的 `master` 分支，GitHub Pages 会自动更新站点。

## 项目结构

```text
.
├── _config.yml                 # 站点配置
├── _config.next.yml            # NexT 主题配置
├── scaffolds/                  # 新文章模板
├── source/
│   ├── _posts/                 # 文章，按分类分目录
│   ├── about/                  # 关于页面
│   ├── categories/             # 分类页面
│   ├── schedule/               # 计划页面
│   └── tags/                   # 标签页面
└── public/                     # 生成的静态文件（部署内容）
```

## 提交约定

提交信息使用 Conventional Commits 风格，常见类型：

- `feat:` 新功能或新页面
- `docs:` 文章或文档更新
- `chore:` 构建、依赖等杂项
- `style:` 样式调整

提交后默认直接推送到远程仓库。
