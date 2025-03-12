---
title: "使用hugo搭建个人博客"
subtitle: ""
description: "在docker容器中搭建hugo开发环境，使用githubactions自动部署到AliyunECS ningx服务器上。"
date: 2024-12-28T14:35:16Z
author:      "Wayne"
image: ""
tags: ["hugo", "blog"]
categories: ["Tech"]
draft: false
---

由于 github pages 在国内访问速度较慢，所以选择阿里云 ECS 服务器 NGINX 托管/host 博客静态资源。
由于国内 github 访问速度较慢，所以代码厂库托管在 gitee 上，然后将 gitee 仓库 自动同步到 github 仓库。

## gitee 仓库同步 github 仓库

省略

## 使用 docker 搭建 hugo 开发环境

1. `docker pull hugomods/hugo`
2. `docker run -itd --name hugo -v $(pwd):/src -p 1313:1313 hugomods/hugo /bin/sh`，这里因为 hugo 容器启动后的默认工作目录是/src,所有我们将 hugo 的 root 目录直接挂载到该位置/src 后，一启动进入容器就可以进入 hugo 的 root 工作目录中。
3. `hugo server -D -M --bind 0.0.0.0 &`查看效果

## 使用 github actions 自动部署到阿里云 ECS 服务器

1. 在 github 仓库中创建 workflow(.github/workflows/hugo.yml),内容如下：

```yml
# 本workflow通过github actions部署hugo static website,并将hugo的public目录内容sync同步到aliyunECS机器的nginx static文件目录中
name: Deploy hugo site by githubactions and Sync public directory into Aliyun ECS disk

on:
  push:
    branches:
      - main
      - master

  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.137.1

    steps:
      - name: install hugo cli
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: install dart sass
        run: sudo snap install dart-sass
      - name: checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify
      - name: upload artfacts
        uses: actions/upload-artifact@v4
        with:
          name: public
          path: public/

  # sync public directory into aliyunECS
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: download artifacts
        uses: actions/download-artifact@v4
        with:
          name: public
          path: public

      - name: sync hugo static public directory into remote AliyunECS by SSH
        uses: easingthemes/ssh-deploy@v2.1.5 #可以访问的仓库，实现的上传服务器步骤被封装在此action
        env:
          SSH_PRIVATE_KEY: ${{ secrets.ALIYUN_SSH_PRIVATE_KEY }} #这个是阿里云的私钥
          ARGS: "-avzr --delete"
          SOURCE: "./public"
          REMOTE_HOST: ${{ secrets.ALIYUN_ECS_HOST }} #阿里云的 ip
          REMOTE_USER: ${{ secrets.ALIYUN_ECS_USER }} #阿里云用户
          TARGET: "/opt/webapp/static/hugo" #NGINX静态文件目录
```

2. 在阿里云 ECS 服务器上创建 SSH 密钥对。并将私钥保存到 github 仓库的 secrets 中，命名为 ALIYUN_SSH_PRIVATE_KEY；同时将公钥保存到阿里云 ECS 服务器上，并添加到(ECS 的)~/.ssh/authorized_keys 文件中。

TIPS:

1. 阿里云 ECS 服务器上需要安装 nginx，并将静态文件目录设置为`/opt/webapp/static/hugo`
2. github 仓库的 Settings->Pages 的 source 需要选择 Github Actions
