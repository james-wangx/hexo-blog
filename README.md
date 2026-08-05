# James Wang's Blog

![Hexo](https://img.shields.io/badge/Hexo-8.1.2-0e83cd)
![NexT](https://img.shields.io/badge/NexT-8.28.0-0e83cd)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Active-222)
![License](https://img.shields.io/badge/License-MIT-blue)

Personal technology blog built with Hexo 8 + NexT 8 and hosted on GitHub Pages.

Online: <https://james-wangx.github.io>

[中文文档](README.zh-CN.md)

## Tech Stack

- Hexo 8 static site generator
- NexT 8 theme
- GitHub Pages deployment
- hexo-generator-searchdb site search
- hexo-generator-sitemap sitemap
- hexo-next-giscus comments
- hexo-relative-link relative image paths

## Local Development

Install dependencies:

```bash
npm install
```

Common commands:

| Command | Description |
| --- | --- |
| `npm run server` | Local preview at http://localhost:4000 |
| `npm run build` | Generate static files into `public` |
| `npm run clean` | Clean generated cache (`public`, `db.json`) |
| `npm run deploy` | Deploy `public` to GitHub Pages |

Full deployment workflow:

```bash
npm run clean
npm run build
npm run deploy
```

## Writing Posts

Posts live under `source/_posts/`, organized into category directories, for example:

```text
source/_posts/PLM/Teamcenter/My-Article.md
```

Writing conventions:

- Use English titles, and replace spaces with `-` in file names.
- Set the category in the front matter, for example `PLM` + `Teamcenter`.
- Put images into the directory named after the post and reference them with relative paths:

```markdown
![alt](./My-Article/image.png)
```

Front matter example:

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

## Deployment

The deployment target is configured in `_config.yml`:

```yaml
deploy:
  type: git
  repo: https://github.com/james-wangx/james-wangx.github.io
  branch: master
```

After running `npm run deploy`, the contents of `public` are pushed to the `master` branch of the `james-wangx.github.io` repository, and GitHub Pages updates the site automatically.

## Project Structure

```text
.
├── _config.yml                 # Site configuration
├── _config.next.yml            # NexT theme configuration
├── scaffolds/                  # Post templates
├── source/
│   ├── _posts/                 # Posts, organized by category
│   ├── about/                  # About page
│   ├── categories/             # Categories page
│   ├── schedule/               # Schedule page
│   └── tags/                   # Tags page
└── public/                     # Generated static files (deployed content)
```

## Commit Convention

Commit messages follow the Conventional Commits style:

- `feat:` new feature or page
- `docs:` post or documentation update
- `chore:` build, dependency, and other maintenance
- `style:` style changes

Changes are pushed to the remote repository after each commit.
