---
layout:     post
title:      "centos7下allabag非docker搭建遇到的问题与解决方法"
subtitle:   "Problems and solutions encountered in non-docker construction of Wallabag under centos7"
date:       2022-07-10 16:45:00
author:     "Lv Hui"
header-img: "img/chrome-bg.png"
header-mask: 0.3
catalog: true
tags:
    - Wallage
---


平时上网的时候有时碰到有用的文章或者网站总是一键收藏在浏览器的收藏夹中，但是时间一长就越积越多。看着那么多收藏反而懒得整理了，而且有的链接在过了一段时间之后就已经失效了，再想起来找的时候又觉得有些可惜。所以现在浏览器的收藏只放一些短期的和常用的链接，收藏的文章和不常用的链接就放在pinbox中，最后还有一些看完觉得还需要保留的文章，所以就准备自己搭建一个网页收藏服务。Wallabag是一个开源免费的自建网页收藏服务，它会把网页的内容保存下来，所以就不怕链接失效了，而且还能为保存的网页生成RSS订阅源，可以随时在自己喜欢的RSS阅读器上阅读。由于当时搭建时服务器上没有安装docker，就直接选择了手动安装，没想到过程中还是遇到了不少的问题。

## 下载过慢的问题

```bash
git clone https://github.com/wallabag/wallabag.git
```

直接从github上有的时候很难直接拉下来，在不使用代理的情况有两种解决方法

1. 使用国内的镜像地址替换上面的github.com，镜像地址自行查找吧，毕竟随时就可能挂掉了。
2. 直接下载下来再传到服务器上，再在服务器上解压。但是可能速度也不快，但是如果你本地有梯子的话就很快了
![github release](/img/in-post/release.png)

## 执行make install遇到 Do not run this script as root!

```bash
[root@VM-4-17-centos wallabag]# make install
Do not run this script as root!
Use --ignore-root-warning to ignore this error.
make: *** [install] Error 1
```

改成执行下面的命令，忽略掉当前是root用户的警告，当然你也可以切换成其他用户来避免这个问题

```bash
./scripts/install.sh prod --ignore-root-warning
```

## 安装时composer.phar install 报错

类型于下面这种

```bash
Problem 1
- Root composer.json requires craue/config-bundle, it could not be found in any version, there may be a typo in the package name.
```

这是因为我之间修改过composer的默认地址为国内镜像，可能是镜像挂掉了吧，所以把默认地址改成packagist的就能解决

```bash
composer config -g repo.packagist composer https://packagist.org
```

## 缺少tidy扩展问题

原因是没有安装php的tidy扩展

```bash
#执行如下命令,我的php是宝塔安装的,所以php的目录需要根据自己的安装位置自行修改
yum install libtidy libtidy-devel -y
cd /www/server/php/74/src/ext/tidy
/www/server/php/74/bin/phpize
./configure --with-php-config=/www/server/php/74/bin/php-config
make && make install
echo "extension = tidy.so" >> /www/server/php/74/etc/php.ini
service php-fpm-74 restart 
```

再次执行时依然提示tidy扩展的问题

原来是php-cli用的是另一个配置文件php-cli.ini，重新修改php-cli.ini

```bash
echo "extension = tidy.so" >> /www/server/php/74/etc/php-cli.ini
service php-fpm-74 restart 
```

## 安装成功后无法打开页面

1. 网站的默认根目录为web文件夹
2. 配置反向代理
3. wallabag只能以域名的方式访问，需要解析域名
4. 修改后记得清理一下浏览器缓存

## 安装时域名或数据库配置错误

可能在安装过程中输入了错误的域名，或者数据库的名称、用户或者密码填写错误。那么可以直接修改/app/config/parameters.yml文件

```bash
#修改配置文件后清理缓存
php bin/console cache:clear --env=prod
```

## 访问时报错权限问题

将项目的所有者由root用户改为www用户，再给文件夹赋予755权限即可解决