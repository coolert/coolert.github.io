---
layout:     post
title:      "Debian11 安装 PHP 运行环境"
subtitle:   "Debian11 install php operating environment"
date:       2024-06-20 13:37:00
author:     "Lv Hui"
header-img: /lsky.coolerk.com/i/2024/06/21/667546a311aca.png
header-mask: 0.7
catalog: true
tags:
    - PHP
---

## 更新apt

```shell
sudo apt update
sudo apt upgrade
```

## 安装并启动nginx

### 通过apt安装nginx

```shell
sudo apt install nginx
```

### 开启nginx服务

```shell
#启动
sudo systemctl start nginx
#设置开机自启
sudo systemctl enable nginx
```

## 安装PHP

apt库默认只有7.4版本的PHP，如果想安装更高的版本需要添加外部存储库来安装更高版本的PHP

### 添加外部存储库

```shell
sudo apt update
sudo apt install -y lsb-release apt-transport-https ca-certificates
sudo wget -qO /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
```

### 更新包列表

```shell
sudo apt update
```

### 安装PHP及扩展

```shell
sudo apt install php8.1 php8.1-cli php8.1-fpm php8.1-mysql php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml php8.1-bcmath
```

## 安装与配置Mysql

apt库中没有mysql，只能先添加Mysql的源。

获取最新版本链接：[https://dev.mysql.com/downloads/repo/apt/](https://dev.mysql.com/downloads/repo/apt/)

### 添加Mysql源

```shell
sudo wget https://dev.mysql.com/get/mysql-apt-config_0.8.30-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.30-1_all.deb
sudo rm mysql-apt-config_0.8.30-1_all.deb
```

### 安装Mysql

```shell
sudo apt update
sudo apt install mysql-server
```

### 新建一个能够远程链接的账户

```sql
CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
