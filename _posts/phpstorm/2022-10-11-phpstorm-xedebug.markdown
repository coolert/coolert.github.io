---
layout:     post
title:      "homestead环境下phpstorm配置xdebug的一些问题"
subtitle:   "Some problems with xdebug configuration in phpstorm in homestead"
date:       2022-10-11 15:29:00
author:     "Lv Hui"
header-img: "img/phpstorm/xdebug/bg.png"
header-mask: 0.3
catalog: true
tags:
    - xdebug
    - phpstorm
---

断点调试其实是非常方便的，所以在本地配置一下，具体的配置流程可以看[PhpStorm + Homestead 配置 xdebug](https://learnku.com/articles/31342)，写的非常详细了，这里只记录一下遇到的问题与解决。

### 提示 waiting for incoming connection with ide key '11985’

php.ini 需要配置xdebug

xdebug3的配置与xdebug2配置会有所不同，具体配置如下

```bash
xdebug.mode = debug
xdebug.client_host = "192.168.10.1"
xdebug.client_port = "9003"
xdebug.collect_return = On
xdebug.idekey = phpstorm
```

### 配置浏览器调试，验证配置是否有问题

点击右上角 Edit Configurations

![Edit Configurations](/img/phpstorm/xdebug/Edit%20Configurations.jpg)

再点击+号添加一个 PHP Web Page

![Create configuration](/img/phpstorm/xdebug/Create%20configuration.png)

Name 可以随意填写，Server选择之前配置过的服务，Start URL填具体的请求地址。然后点击Validate验证配置是否正确。

![PHP Web Page](/img/phpstorm/xdebug/PHP%20Web%20Page.png)

![Validate](/img/phpstorm/xdebug/Validate.png)

点击下方Validate按钮，检测配置是否正确，有错误的话会有提示，无错误时入下图。

![Result](/img/phpstorm/xdebug/Result.png)