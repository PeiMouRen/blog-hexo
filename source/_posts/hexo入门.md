---
title: hexo入门
date: 2023-03-09 17:07:36
categories: hexo
tags: hexo
---

[toc]

# 简介

hexo是一个轻量的静态博客生成工具，可配合`github`等构建自己的博客。

官网：`https://hexo.io/zh-cn/index.html`

# 安装

> 安装`hexo`前先在电脑上安装好`git`和`node.js`。

```bash
npm install -g hexo-cli
```

# 建站

```bash
# 在folder目录创建网站
# 注意建站的文件夹必须是空文件夹，如果不是空文件夹，需要先清空，建站完毕后再将自己的文件拷贝进来
hexo init [folder]
npm install
# 查看hexo版本
hexo version
```

建站完成后会生成相应的文件和目录：

## _config.yml

网站的配置信息

## source

资源文件夹，除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。

## themes

主题文件夹，`Hexo`会根据不同的主题来生成不同的静态页面。

# 配置

参考官网各配置项。

# 命令

## help

> 命令帮助

```
hexo -h
```

## new

```
# 使用默认的post布局创建博客，-p指定博客路径
hexo new [title] -p [path]
```

## generate

> 用md文件生成静态文件（html），静态文件放在`public`目录下。

```
# 可简写为hexo g
hexo generate
# -d部署博客网站
hexo g -d
```

## deploy

> 部署博客网站。

```
# 可简写为hexo d
hexo deploy
# 同hexo g -d
hexo d -g
```

## server

> 本地启动博客网站，启动后默认访问地址为`http://localhost:4000/`。

```
hexo server
```

## clean

> 清除缓存`db.json`和已生成的静态文件`public`。

```
hexo clean
```

# 部署到github

> 即执行`hexo deploy`时，将静态资源推送到`github`的`pages`上。
>
> 前提是需要有开通pages的github仓库。

## github pages

> 就是github免费将你一个仓库里的代码部署到公网上（大概是这个意思），前提是仓库名需要为`github account.github.io`。
>
> 例如我的github账号为`xiaoming`，那么我可以新建一个名为`xiaoming.github.io`的仓库，之后配置好这个仓库的`pages`，再用浏览器访问`https://xiaoming.github.io`就可以访问到自己的博客网站了。

## 安装插件

```
npm install hexo-deployer-git --save
```

## 添加配置

在`_config.yml`中配置

```
deploy:
  type: git
  repo: https://github.com/PeiMouRen/peimouren.github.io.git
  branch: master
  token: ghp_4pBN123xjGo4pi2Q6MOf1234RXSq168Vlyg9M4JmxLT

```

# 问题

## 部署到github时身份验证问题

采用hexo的一键部署时，若在`_config.yml`中配置的`github`的`token`还是提示输入用户名和密码时，这里的用户名随便输，密码需为`github`的`token`。

`github`设置token：`头像 -> setting -> Developer setting -> Personal access tokens`.

## 新建的博客md文件头上没有categories、tags信息

```bash
hexo new page categories
```

打开`source/categoriex/index.md`，在顶部`title`和`date`下加上`type: categories`。

再次创建博客md文件时，头上就会出现`categories`信息。

`tags`的创建同`categories`。

