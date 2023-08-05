---
layout:     post
title:      "Laravel sail环境下phpstorm配置xdebug"
subtitle:   "Phpstorm configuration xdebug in laravel sail environment"
date:       2023-08-05 15:29:00
author:     "Lv Hui"
header-img: "img/phpstorm/xdebug/bg.png"
header-mask: 0.3
catalog: true
tags:
    - xdebug
    - phpstorm
---

## 修改`.env`文件

在laravel的`.env`文件中加入一行xdebug配置`SAIL_XDEBUG_MODE=develop,debug,coverage`，然后重启laravel sail使配置生效

## phpstorm配置php解释器

设置->PHP 选择通过docker配置

![解释器1](/img/sail_debug/config_interpreters_1.png)

服务选择laravel.test

![解释器2](/img/sail_debug/config_interpreters_2.png)

最后点击确认就配置好了

![解释器3](/img/sail_debug/config_interpreters_3.png)

## 配置项目目录映射

![目录映射](/img/sail_debug/file_map.png)

## 发送请求

### 浏览器插件

chrome 浏览器下载Xdebug helper插件

[Xdebug helper 商店地址](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc)

### postman配置

有两种方式可以选择

1. 直接在请求地址加上参数`XDEBUG_SESSION_START=phpstorm`

2. 在cookie里加上`XDEBUG_SESSION=PHPSTORM`

   也可以为指定域名配置cookie,这样就不用对每一个api单独设置了

   ![postman cookie](/img/sail_debug/postman_cookie.png)

## 开启phpstorm侦听

![侦听按钮](/img/sail_debug/listen.png)

发送请求后,断点的debug信息

![success](/img/sail_debug/success.png)





