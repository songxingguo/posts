---
title: 使用Github Action部署静态网站
description: 使用Github Action部署静态网站
urlname: tfub27hk86lsdrpb
date: 2023-01-15 12:05:37 +0000
tags:
  - 场景/博客
categories:
  - 前端
---
## 目录

## 生成 token

1. 进入 Github 生成[授权 token 的页面](https://github.com/settings/apps)，创建一个 token，修改名称并选择所有仓库。

![31b1d2e711f2758b0d726f3722f53848 MD5](https://image.songxingguo.com/obsidian/20240714/31b1d2e711f2758b0d726f3722f53848_MD5.png)

2. 将仓库权限设置中的 Content 项，设置为读和写。

![5e1fff853aed8142210b7cdda7e1680d MD5](https://image.songxingguo.com/obsidian/20240714/5e1fff853aed8142210b7cdda7e1680d_MD5.png)

3. 最后点击确认生成 token，并**复制**，**注意 ⚠️ 刷新之后就没有办法复制了**。

![28a0735784c9b88bc187424f0232dace MD5](https://image.songxingguo.com/obsidian/20240714/28a0735784c9b88bc187424f0232dace_MD5.webp)

## 设置环境变量

1. 进入到**项目**设置页面，选择添加[Actions 的环境变量](https://github.com/songxingguo/resume/settings/secrets/actions)。

![2de454c076113f81848c9e68b5031bf5 MD5](https://image.songxingguo.com/obsidian/20240714/2de454c076113f81848c9e68b5031bf5_MD5.webp)

2. 输入变量名称`ACCESS_TOKEN`，密钥一栏粘贴刚才复制的 Token，点击添加就完成了。

![d730907c769fae0040d94aba7bc65070 MD5](https://image.songxingguo.com/obsidian/20240714/d730907c769fae0040d94aba7bc65070_MD5.webp)

## 项目中添加配置文件

1. 在项目的更目录下创建`.github/workflows/static.yml`。

```javascript
name: GitHub Actions Build and Deploy
on:
  push:
    branches:
      - main #触发自动化部署的分支
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️ #从触发分支检出代码
        uses: actions/checkout@v3
        with:
          persist-credentials: false

      - name: Install and Build 🔧 #安装依赖并且打包静态文件
        run: |
          npm i
          npm run build
      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BRANCH: gh-pages #部署的分支
          FOLDER: dist #打包后静态文件所在的位置
          CLEAN: true
```

2. 然后提交代码就开始自动部署了。

![4ba5e06792e715e734a7fbff02f8946e MD5](https://image.songxingguo.com/obsidian/20240714/4ba5e06792e715e734a7fbff02f8946e_MD5.webp)
