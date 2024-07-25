---
layout:     post
title:      "Laravel Sail 配置mongodb"
subtitle:   "Laravel Sail configures mongodb"
date:       2024-07-25 14:35:00
author:     "Lv Hui"
header-img: "/lsky.coolerk.com/i/2024/07/25/66a1f264ce2b1.webp"
header-mask: 0.6
catalog: true
tags:
    - PHP
---

Laravel Sail 默认不支持mongodb，但是由于它本身就是基于docker的，所以我们手动修改`docker-compose.yml`文件

## 修改 docker-compose.yml

```yml
# For more information: https://laravel.com/docs/sail
version: '3'
services:
    laravel.test:
        build:
            context: ./docker/8.2
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.2/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mysql
            - redis
            - mongodb # 依赖增加mongodb
            - meilisearch
    mysql:
        image: 'mysql/mysql-server:8.0'
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ROOT_HOST: "%"
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 1
        volumes:
            - 'sail-mysql:/var/lib/mysql'
            - './vendor/laravel/sail/database/mysql/create-testing-database.sh:/docker-entrypoint-initdb.d/10-create-testing-database.sh'
        networks:
            - sail
        healthcheck:
            test: ["CMD", "mysqladmin", "ping", "-p${DB_PASSWORD}"]
            retries: 3
            timeout: 5s
    # 增加mongodb服务
    mongodb:
        image: 'mongo:latest' # 使用的镜像
        ports:
            - '27017:27017'
        environment:
            MONGO_INITDB_ROOT_USERNAME: '${DB_USERNAME}'
            MONGO_INITDB_ROOT_PASSWORD: '${DB_PASSWORD}'
        volumes:
            - 'sail-mongodb:/data'
        networks:
            - sail
        healthcheck:
            test: test $$(echo "db.runCommand('ping').ok" | mongo --quiet --host localhost --port 27017) -eq 1
            timeout: 10s
            retries: 5
    redis:
        image: 'redis:alpine'
        ports:
            - '${FORWARD_REDIS_PORT:-6379}:6379'
        volumes:
            - 'sail-redis:/data'
        networks:
            - sail
        healthcheck:
            test: ["CMD", "redis-cli", "ping"]
            retries: 3
            timeout: 5s
    meilisearch:
        image: 'getmeili/meilisearch:latest'
        ports:
            - '${FORWARD_MEILISEARCH_PORT:-7700}:7700'
        volumes:
            - 'sail-meilisearch:/meili_data'
        networks:
            - sail
        healthcheck:
            test: ["CMD", "wget", "--no-verbose", "--spider",  "http://localhost:7700/health"]
            retries: 3
            timeout: 5s
networks:
    sail:
        driver: bridge
volumes:
    sail-mysql:
        driver: local
    sail-redis:
        driver: local
    sail-meilisearch:
        driver: local
    #数据卷，持久化数据
    sail-mongodb:
        driver: local
```

## 修改laravel.test容器的构建

执行以下命令发布配置文件，会在根目录生成一个docker目录，里面就有各个php版本的配置。发布后会自动更改docker-compose.yml中laravel.test的引用目录。

```shell
sail artisan sail:publish
```

![docker目录](https://lsky.coolerk.com/i/2024/07/25/66a1f991e657c.png)

修改Dockerfile，在`php8.2-xdebug`后面增加`php8.2-mongodb`
然后执行

```shell
sail down
sail build -no-cache
```

这里可能会失败，由于我的电脑是mac的m1芯片，mongodb扩展所需的依赖没有在ubuntu的官方源中提供arm64的版本，所以可能会提示404，并且构建失败。
如果和我有同样的问题，可以先安装`php-pear`再用`pecl`安装mongodb扩展

以下是完整的Dockefile

```shell
FROM ubuntu:22.04

LABEL maintainer="Taylor Otwell"

ARG WWWGROUP
ARG NODE_VERSION=18
ARG POSTGRES_VERSION=14

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 dnsutils \
    && curl -sS 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c' | gpg --dearmor | tee /usr/share/keyrings/ppa_ondrej_php.gpg > /dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php8.2-cli php8.2-dev \
       php8.2-pgsql php8.2-sqlite3 php8.2-gd \
       php8.2-curl \
       php8.2-imap php8.2-mysql php8.2-mbstring \
       php8.2-xml php8.2-zip php8.2-bcmath php8.2-soap \
       php8.2-intl php8.2-readline \
       php8.2-ldap \
       php8.2-msgpack php8.2-igbinary php8.2-redis php8.2-swoole \
       php8.2-memcached php8.2-pcov php8.2-xdebug \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarn.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && curl -sS https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /usr/share/keyrings/pgdg.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update \
    # 安装 php-pear
    && apt-get install -y php-pear \
    # 安装 mongodb扩展
    && pecl install mongodb \
    && apt-get install -y yarn \
    && apt-get install -y mysql-client \
    && apt-get install -y postgresql-client-$POSTGRES_VERSION \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 将扩展写入配置文件，开启扩展
RUN echo "extension=mongodb.so" > /etc/php/8.2/cli/conf.d/mongodb.ini

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.2

RUN groupadd --force -g $WWWGROUP sail
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 sail

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/8.2/cli/conf.d/99-sail.ini
RUN chmod +x /usr/local/bin/start-container

EXPOSE 8000

ENTRYPOINT ["start-container"]

```