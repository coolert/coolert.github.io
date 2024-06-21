---
layout:     post
title:      "Debian11 安装 php imagick 扩展"
subtitle:   "Debian11 install php imagick extension"
date:       2024-06-20 18:26:00
author:     "Lv Hui"
header-img: "/lsky.coolerk.com/i/2024/06/21/667546a311aca.png"
header-mask: 0.7
catalog: true
tags:
    - PHP
---

## 安装步骤

1. 更新包列表并安装依赖 

    安装 imagemagick 和 php-pear

    ```shell
    sudo apt update
    sudo apt install imagemagick php-pear
    ```

2. 安装 PHP8.1 的开发包

    ```shell
    sudo apt install php8.1-dev
    ```
3. 使用 PECL 安装 Imagick

    ```shell
    sudo pecl install imagick
    ```
    安装中的选项可以直击回车使用默认值。
    
    如果出现以下错误，是因为系统找不到 MagickWand-config 或 Wand-config 程序
    ```
    configure: error: not found. Please provide a path to MagickWand-config or Wand-config program.
    ERROR: `/tmp/pear/temp/imagick/configure --with-php-config=/usr/bin/php-config --with-imagick' failed
    ```
    
    为了解决这个问题可以安装MagickWand库，然后再重新安装Imagick扩展

    ```shell
    sudo apt install libmagickwand-dev
    ```
   
4. 启用 Imagick 扩展
   
    ```shell
    sudo bash -c "echo 'extension=imagick.so' > /etc/php/8.1/mods-available/imagick.ini"
    sudo phpenmod imagick
    ```

5. 重启 Web 服务器

    ```shell
    sudo systemctl restart php8.1-fpm
    sudo systemctl restart nginx
    ```

6. 检查是否已成功加载
    
    ```shell
    php -m | grep imagick
    ```

    如果输出中包含 imagick，则说明安装成功。