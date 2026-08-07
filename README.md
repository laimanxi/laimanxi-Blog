# laimanxi-Blog

我的个人博客源码，使用 [Hexo](https://hexo.io) 构建，主题为 [Redefine](https://github.com/EvanNotFound/hexo-theme-redefine)，部署于 GitHub Pages。

- 线上地址：https://laimanxi.github.io
- 部署仓库：https://github.com/laimanxi/laimanxi.github.io（由 `hexo deploy` 自动管理，勿手动修改）

## 文档

- [首页横幅特效与全站背景实现文档](docs/homepage-effects.md) —— 晕染圆球背景、鼠标揭示文字的原理与排障备忘

## 仓库结构

```
├── _config.yml            # Hexo 站点配置（标题、URL、部署等）
├── _config.redefine.yml   # Redefine 主题覆盖配置（改主题样式只动这里）
├── _config.fluid.yml      # 旧主题 Fluid 的覆盖配置（留档，未生效）
├── source/
│   ├── _posts/            # 博客文章（Markdown）
│   ├── about/             # 关于页
│   ├── tags/              # 标签索引页
│   ├── categories/        # 分类索引页
│   └── img/               # 自定义图片（favicon 等）
├── scaffolds/             # 新文章/页面的模板
└── themes/fluid           # 旧主题（git 子模块，留档备用）
```

## 日常写作流程

```bash
# 1. 新建文章（生成到 source/_posts/）
npx hexo new "文章标题"

# 2. 本地预览（http://localhost:4000，改动自动刷新）
npx hexo server

# 3. 部署上线（自动生成 + 推送到 laimanxi.github.io）
npx hexo deploy -g

# 4. 备份源码到 GitHub
git add .
git commit -m "Add post: 文章标题"
git push
```

## 文章 front-matter 规范

```markdown
---
title: 文章标题
date: 2026-08-05 12:00:00
tags:
  - 学习笔记
categories:
  - 技术
---

这里是摘要，会显示在首页列表。

<!--more-->

这里是正文……
```

`<!--more-->` 以上是首页摘要，以下内容需点开文章查看。

## 文章中插入图片

已开启 `post_asset_folder`：用 `npx hexo new "文章名"` 创建文章时，会同时生成同名资源文件夹，图片放进该文件夹，用相对路径引用：

```
source/_posts/
├── my-post.md
└── my-post/          ← 同名文件夹
    └── photo.jpg
```

```markdown
![图片描述](photo.jpg)
```

- 图片文件名不要用中文和空格（如 `photo-01.jpg`），注意扩展名大小写
- 全站通用图片（头像、logo 等）放 `source/img/`，用绝对路径引用：`![logo](/img/logo.png)`
- 建议单张图片压缩到几百 KB 以内，避免页面加载慢

## Hexo 常用命令

```bash
npx hexo new draft "草稿标题"   # 新建草稿（source/_drafts/，不发布）
npx hexo publish "草稿标题"     # 草稿转为正式文章
npx hexo new page "页面名"      # 新建独立页面

npx hexo clean     # 清空缓存和 public/（页面不更新时先用它）
npx hexo generate  # 只生成静态文件，不部署
npx hexo list post # 列出所有文章
```

> 口诀：页面显示不正常 → 先 `hexo clean` 再重新生成。

## Git 常用命令

```bash
git status              # 查看改动
git add .               # 暂存所有改动
git commit -m "说明"    # 提交
git push                # 推送到本仓库

git log --oneline       # 提交历史
git diff                # 查看未暂存的改动
```

## 换电脑后恢复环境

```bash
# --recurse-submodules 必加，否则 themes/fluid 目录是空的
git clone --recurse-submodules https://github.com/laimanxi/laimanxi-Blog.git
cd laimanxi-Blog
npm install
```

## 两个仓库的区别

| 操作 | 目标仓库 | 作用 |
|---|---|---|
| `git push` | laimanxi-Blog | 备份**源码**（文章 md、配置） |
| `npx hexo deploy` | laimanxi.github.io | 发布**生成的网页**（线上实际内容） |

发新文章后两个都要执行。

## 常见问题

| 场景 | 操作 |
|---|---|
| 改主题样式/功能 | 编辑 `_config.redefine.yml` |
| 改站点标题/描述 | 编辑 `_config.yml` |
| 删文章 | 删除 `source/_posts/` 对应文件，再 `npx hexo deploy -g` |
| 部署后线上没变化 | 等约 1 分钟（Pages 构建延迟）；仍无效则 `hexo clean` 后重新 `deploy -g` |
| 升级主题 | `npm update hexo-theme-redefine` |
