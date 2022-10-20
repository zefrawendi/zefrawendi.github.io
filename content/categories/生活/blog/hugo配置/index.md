---
title: "hugo配置"
date: 2022-10-19T15:00:00+08:00
lastmod: 2021-06-02T18:25:17+08:00
draft: false
image: "cover.png"
description: "使用hugo搭建个人博客"
categories: ["开发"]
tags: ["hugo","blog"]
slug: "hugo"
---


## 安装hugo并配置主题

### 安装hugo
```shell
brew install hugo
```

### 创建hugo项目
```shell
hugo new site zefrawendi.github.io
```

### 添加主题
```shell
cd zefrawendi.github.io
git init
git submodule add 
```

### 添加内容
```shell
hugo new post/others/blog/index.md
```

### 本地预览
```shell
hugo server -D
```

## github pages配置

### 创建github pages项目
```shell
zefrawendi.github.io
```

### 添加github pages项目为远程仓库
```shell
git remote add origin
```

### 添加github pages项目为子模块
```shell
git submodule add -b master
```

## 博客写法

### 添加文章
```shell
hugo new post/others/blog/index.md
```

### 相关配置
```shell
---
title: "hugo stack配置"              // 文章标题
date: 2022-10-19T14:49:05+08:00     // 创建时间
draft: true                         // 是否发布
image: "conver.png"                 // 封面图
description: "hugo stack配置"        // 描述
categories: ["others"]              // 分类
tags: ["hugo"]                      // 标签
slug: "hugo-stack"                  // 用于生成文章链接
---
```