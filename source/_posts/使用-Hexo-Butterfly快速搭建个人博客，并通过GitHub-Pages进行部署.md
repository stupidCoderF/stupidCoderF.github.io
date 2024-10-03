title: 使用 Hexo + Butterfly快速搭建个人博客，并通过GitHub Pages进行部署
author: 莫道君行早
tags:
  - Hexo
categories: []
date: 2024-10-03 18:39:00
---
# 使用 Hexo + Butterfly快速搭建个人博客，并通过GitHub Pages进行部署

## 准备工作

### 安装Node.js

(Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)
[查看具体版本限制](https://hexo.io/zh-cn/docs/#Node-js-%E7%89%88%E6%9C%AC%E9%99%90%E5%88%B6)

[下载地址](https://nodejs.org/en)

### 安装Git

[下载地址](https://git-scm.com/downloads)

### 注册Github账号

此步骤自行搜索

## 1. 使用Hexo初始化项目

### 安装Hexo脚手架，并初始化项目

```shell
# 方案一：全局安装hexo脚手架

npm install -g hexo-cli 
hexo init <文件夹> # 如不指定文件夹，则在当前目录下初始化项目
cd <文件夹>
npm install


# 方案二：使用npx（推荐）

npx hexo init # npx会局部安装hex，并自动安装依赖，无需执行npm install
```

完成以上操作后文件夹中会有以下结构：

```
  ├── _config.yml
  ├── package.json
  ├── scaffolds
  ├── source
  |   ├── _drafts
  |   └── _posts
  └── themes
```

### 预览项目

```shell
npx hexo server
```

## 2. 将代码提交到Github

- 在GitHub中新建名为 <Github 用户名>.github.io 的仓库
- 将仓库设置为公开
- 将代码上传至仓库

```shell
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:<你的 GitHub 用户名>/<你的 GitHub 用户名>.github.io.git
git push -u origin main
```

## 3. 将Hexo的默认主题更改为Butterfly

### 下载Butterfly主题

```shell
# 移除hexo的默认主题
npm uninstall hexo-theme-landscape
# 添加butterfly主题
git submodule add -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

```shell
# 出现该错误时，可以检查git代理是否配置正确
fatal: unable to access 'https://github.com/jerryc127/hexo-theme-butterfly.git/': OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 0
fatal: clone of 'https://github.com/jerryc127/hexo-theme-butterfly.git' into submodule path 'D:/Project/hexo/themes/butterfly' failed

# 执行以下命令查看git配置
git config --global -l

# 如果出现 http.proxy 或 https.proxy 执行以下命令：
# 取消http代理
git config --global --unset http.proxy
# 取消https代理 
git config --global --unset https.proxy

# 将端口改为自己的代理端口
git config --global http.proxy http://127.0.0.1:端口
```

### 配置主题

1. 将项目根目录下的_config.yml，把主题替换为butterfly

```yaml
theme: butterfly
```

2. 将/themes/butterfly/_config.yml复制到/_config.butterfly.yml

3. 安装插件

```shell
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

4. 将修改提交至远程仓库

## 4. 配置Github Pages，部署项目

- 在GitHub仓库页面点击Setting > Pages

```shell
# 出现该提示，请先将仓库设为公开
Upgrade or make this repository public to enable Pages
```

- 点击Source下的多选框，选择Github Actions
- 创建配置文件

创建.github/workflows/pages.yml文件，写入以下内容后保存

***将node-version改为与本地Node主版本一致***

本地执行`node -v`可查看Node版本

```yaml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20.x
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```
等待部署完成，Setting > Pages页面出现Your site is live at https://你的GitHub用户名.github.io/  

然后就可以通过 https://你的GitHub用户名.github.io/ 访问了
