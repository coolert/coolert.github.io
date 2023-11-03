---
layout:     post
title:      "宝塔配置php的curl支持HTTP/2"
subtitle:   "Configure php curl to support HTTP/2 in baota panel"
date:       2023-11-03 14:39:00
author:     "Lv Hui"
header-img: "img/bg/http-2.png"
header-mask: 0.3
catalog: true
tags:
    - PHP
    - HTTP/2
---

宝塔默安装的php中curl没有对HTTP/2的支持，所以在需要发送HTTP/2请求时无法满足，重新编译安装php加入对HTTP/2的支持。

## 安装nghttp2

nghttp2是一个用C实现HTTP/2的库

安装的具体要求: [https://github.com/nghttp2/nghttp2#requirements](https://github.com/nghttp2/nghttp2#requirements)

```bash
#安装依赖
yum install libev-devel libevent-devel c-ares-devel jemalloc-devel jansson-devel python-devel zlib-devel
#git安装
git clone https://github.com/nghttp2/nghttp2.git
cd nghttp2
git submodule update --init
autoreconf -i
autoconf
./configure
make
make install
#查看nghttpx版本
nghttpx -v
```

![看nghttpx版本](/img/in-post/php-http2/nghttp_version.png)

### 安装过程中的一些问题

#### 编译报错 error: #error This file requires compiler and library support for the ISO C++ 2011 standard. This support is currently experimental and must be enabled with the -std=c++11 or -std= gnu Q ++11 compiler options.

出现这个错误的原因是编译器版本过旧，可以通过安装devtoolset的方式来更新gcc。devtoolset-7对应gcc7.x.x版本，devtoolset-8对应gcc8.x.x版本，以此类推。这里以devtoolset-8为例。

```bash
#安装centos-release-scl
sudo yum install centos-rease-scl
#安装devtoolset
sudo yum install devtoolset-8
#激活对应版本
scl enable devtoolset-8 bash
#查看gcc版本
gcc -v
#如果没生效执行以下命令
source /opt/rh/devtoolset-8/enable
```

![查看gcc版本](/img/in-post/php-http2/gcc_version.png)

#### 编译错误 undefined reference to 'TLS_client_method'

openssl的版本太低，更新一下openssl

```bash
#下载
https://www.openssl.org/source/openssl-1.1.1l.tar.gz
tar -xvf openssl-1.1.1l.tar.gz
#安装
cd openssl-1.1.1l
./config
make
make install
#配置
echo '/user/local/lib64' >> /etc/ld.so.conf
ldconfig
#备份旧版本
mv /usr/bin/openssl /usr/bin/openssl.old
#建立软链接
ln -s /usr/local/bin/openssl /usr/bin/openssl
ln -s /usr/local/include/openssl /usr/include/openssl
#静态库链接
ln -sf /usr/local/lib64/libcrypto.so.1.1 /usr/lib64/libcrypto.so
ln -sf /usr/local/lib64/libssl.so.1.1 /usr/lib64/libssl.so
#查看版本(也要确认Library的版本)
openssl version
```

![查看openssl版本](/img/in-post/php-http2/openssl_version.png)

#### yum命令报错 

```bash
File "/bin/yum", line 30
except KeyboardInterrupt, e:
^
SyntaxError: invalid syntax
```

yum需要python2，使用python3的需要先将版本切回python2

```bash
#切换python到2.7版本
ln -sf /usr/bin/python2.7 /usr/bin/python
#切换到新的3.8版本
ln -sf /usr/local/bin/python3.8 /usr/bin/python
```

## 安装python

nghttp2需要python3.8以上版本，这里以3.8.8为例。如果已安装python3.8以上版本，则跳过本部分

```bash
#下载
wget https://www.python.org/ftp/python/3.8.8/Python-3.8.8.tgz
tar -zxvf  Python-3.8.8.tgz
#编译安装
cd Python-3.8.8
./configure
make
make install
#建立软链接
ln -s /usr/local/bin/python3.8 /usr/bin/python
ln -s /usr/local/bin/pip3.8 /usr/bin/pip
#查看版本是否生效
python -V
```

![查看python版本](/img/in-post/php-http2/python_version.png)

## 安装curl

```bash
#下载
wget https://curl.haxx.se/download/curl-7.74.0.tar.gz
#解压
tar -zxvf curl-7.74.0.tar.gz
cd curl-7.74.0
#配置时指定nghttp的安装路径
.configure --with-nghttp2=/usr/local --with-ssl
make
make install
```

## 重新安装php

选择编译安装，添加自定义模块。模块参数: `-with-curl=/usr/local/bin/curl`，记得与curl的安装目录保持一致

![php模块](/img/in-post/php-http2/php_model.png)

安装完成后执行 `php --ri curl|grep HTTP2`查看是否开始HTTP/2

![php_cli](/img/in-post/php-http2/php_cli.png)

在phpinfo()中查看是否开启HTTP/2

![php_cli](/img/in-post/php-http2/php_fpm.png)