---
layout: post
title: 一人搞定小程序全栈开发--线上运行的小程序源码分享
date: 2018-09-11 09:08:00 +0800
categories: 小程序
tag: 教程
---

-   content
    {:toc}

## 前言

作为小程序入门系列第二篇，这次分享绝对干货拉满，直接开源一个线上已经在运行的个人小程序，通过阅读源码，学习如何利用腾讯云搭建自己的个人小程序。

## 源码，欢迎 star~

[拼词乐斗 GitHub](https://github.com/viewerw/cw-node)

## 项目介绍

**拼词乐斗**是一款类似 H5 游戏的小程序，玩家需要在 10s 内根据单词的释义，将顺序错乱的字母拼成一个完整的单词，这是最初的核心设计理念，但是后来觉得，这样的游戏太枯燥，于是就加入了音乐的概念，让用户在拼词的过程中可以听到音乐，这样加大了趣味性，如果只是做一个单机版的小程序，这就够了，但是，我又希望用户在拼词完成后可以看到自己的世界排名，或者至少可以看到排名靠前的人的歌曲挑战分，于是，就必须要有一个后台服务了，这时候，就开始考虑着手利用腾讯云搭建自己的小程序后台服务（现在也可以考虑使用腾讯云服务，而不是自己搭建服务）。

项目结构：

```
├── README.md
├── client
│   ├── README.md
│   ├── build
│   ├── config
│   ├── dist
│   ├── index.html
│   ├── node_modules
│   ├── package.json
│   ├── project.config.json
│   ├── src
│   ├── static
│   └── yarn.lock
├── project.config.json
└── server
    ├── README.md
    ├── app.js
    ├── config.js
    ├── controllers
    ├── middlewares
    ├── mysql
    ├── node_modules
    ├── nodemon.json
    ├── package-lock.json
    ├── package.json
    ├── process.prod.json
    ├── qcloud.js
    ├── routes
    ├── tools
    └── tools.md
```

### client 小程序端

小程序开发选择了 mpvue 框架，关于为什么使用 mpvue 可以去我的第一篇文章里面查看。mpvue 生成的模板上稍作一些修改后，就可以使用了。

小程序页面：

```
├── err-words           错词展示页面
├── fight               拼词游戏页面
├── friend-room         好友对战房间（待完成）
├── index               首页
├── intro               规则介绍页
├── login               登录页
├── personal-center     个人中心
├── rank                排行榜页面
├── result              游戏结果页
└── songs               歌曲选择页
```

#### 登录页

使用官方提供的[wafer2-client-sdk](https://github.com/tencentyun/wafer2-client-sdk)库的登录功能实现小程序的用户登录。

#### 首页

可以跳转各个功能页面，实现体力银行的功能。

#### 歌曲选择页

选择歌曲，实现分享解锁的功能（由于微信取消了分享的回调，所以采用 hack 方式实现）

#### 游戏中页面

实现拼词的主要逻辑，包括首字母提示，两秒提示，拼错词减时间条等等。

#### 结果页

展示游戏结果，计算得分，更新本地歌曲分数记录，破纪录则调用上传分数接口。

#### 排行榜

调用后台接口展示排行榜，可以切换歌曲

### server 端

项目后台框架来自[Wafer2 快速开发 demo](https://github.com/tencentyun/wafer2-quickstart-nodejs/blob/master/README.md)

使用到的技术栈 nodejs+koa+knex+mysql，对于前端 er 来说这是十分便捷的方式来开发后台服务。

项目中，后台服务新增了两个接口：上传分数，查询排行榜。如何新增路由（接口）可以查看 wafer 库的相关文档，十分详细的指导教程。

可以使用 wafer 提供的信道 tunnel 服务（websocket）实现多人对战的逻辑（doing）。

controller:

```
.
├── index.js
├── login.js
├── message.js
├── rank.js
├── score.js
├── tunnel.js
├── upload.js
└── user.js
```

### 资源存储

小程序开通后台服务后，腾讯云账号下会有免费的对象存储服务，所有的静态资源可以放在里面（比如歌曲，图片）。

### 最后

如果你有兴趣开发一个全栈的小程序，这份源码或许可以帮助你。

[拼词乐斗 GitHub](https://github.com/viewerw/cw-node)

![](/styles/images/cw.jpg)
![](/styles/images/cw.gif)
