---
layout:     post
title:      "Github自动化部署laravel项目到国内服务器"
subtitle:   "Continuous deploy laravel project from github to server in china"
date:       2023-06-17 11:34:00
author:     "Lv Hui"
header-img: "img/bg/blog-continuous-deployment.png"
header-mask: 0.5
catalog: true
tags:
- PHP
- GitHub
---

代码库在github上，服务器用的腾讯云，鉴于网络原因，先使用github action将代码push到Coding上的版本库，再使用Coding上的持续集成进行构建部署。

## Github配置

### 获取/设置 Personal access tokens

先设置access token，位置在 `setting → Developer settings → Personal access tokens`，生成后后记住token与token名称，之后会用到，token仅在生成时能看到，忘记了就只能重新生成了

### 设置Coding账号与密码

在github项目的设置中配置Coding账号密码，位置在`Settings → Secrets and variables → Actions → New repository secret`

- CODING_USERNAME 存放 Coding 的账号；
- CODING_PASSWORD 存放 Coding 帐号的密码；

### 编写workflow

在项目根目录下创建配置文件`.github/workflows/sync.yml`，并在文件中写入下面的代码

```yml
name: Sync-To-Coding

on:
  push:
    branches:
      - main

jobs:
  push-to-mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Clone
        run: |
          git init
          git remote add origin https://${TOKEN_NAME}:${TOKEN}@github.com/${GITHUB_REPOSITORY}.git
          git fetch --all
          for branch in `git branch -a | grep remotes | grep -v HEAD`; do
            git branch --track ${branch##*/} $branch
          done
        env:
          # github上的代码库
          GITHUB_REPOSITORY: ****/****
          # token名
          TOKEN_NAME: *******
          # access_token
          TOKEN: ***************

      - name: Push to Coding
        run: |
          remote_repo="https://${CODING_USERNAME}:${CODING_PASSWORD}@e.coding.net/${CODING_REPOSITORY}.git"

          git remote add coding "${remote_repo}"
          git show-ref # useful for debugging
          git branch --verbose

          # publish all
          git push --all --force coding
          git push --tags --force coding
        env:
          # Coding上的代码库
          CODING_REPOSITORY: ****/****/****
          CODING_USERNAME: ${\{secrets.CODING_USERNAME}}
          CODING_PASSWORD: ${\{secrets.CODING_PASSWORD}}
```

## Coding配置

先在Coding上创建好代码库，记得和workflow中配置的代码库保持一致

创建一个新的持续集成，jenkensfile配置如下

```groovy
pipeline {
  agent any
  stages {
    stage('检出') {
      steps {
        checkout([$class: 'GitSCM',
        branches: [[name: GIT_BUILD_REF]],
        userRemoteConfigs: [[
          url: GIT_REPO_URL,
          credentialsId: CREDENTIALS_ID
        ]]])
      }
    }

    stage('安装依赖') {
      steps {
        sh 'update-alternatives --set php /usr/bin/php8.0'
        sh 'composer install'
        sh 'tar -zcf code.tar.gz *'
      }
    }

    stage('部署到远端服务') {
      steps {
        script {
          def remoteConfig = [:]
          remoteConfig.name = "my-remote-server"
          remoteConfig.host = "${REMOTE_HOST}"
          remoteConfig.port = "${REMOTE_SSH_PORT}".toInteger()
          remoteConfig.allowAnyHosts = true

          withCredentials([
            sshUserPrivateKey(
              credentialsId: "${REMOTE_CRED}",
              keyFileVariable: "privateKeyFilePath"
            )
          ]) {
            // SSH 登录用户名
            remoteConfig.user = "${REMOTE_USER_NAME}"
            // SSH 私钥文件地址
            remoteConfig.identityFile = privateKeyFilePath
            //文件上传
            sshPut(
              remote: remoteConfig,
              // 本地文件或文件夹
              from: './code.tar.gz',
              // 远端文件或文件夹
              into: '/tmp'
            )
            //开启维护模式
            sshCommand(
              remote: remoteConfig,
              command: "cd /www/wwwroot/${PROD_FILE_NAME} && php artisan down"
            )
            //更新项目文件夹文件
            sshCommand(
              remote: remoteConfig,
              command: 'mkdir -p /tmp/code'
            )
            sshCommand(
              remote: remoteConfig,
              command: 'tar -zxf /tmp/code.tar.gz -C /tmp/code'
            )
            //PROD_FILE_NAME 为环境变量，是具体项目的目录名称
            sshCommand(
              remote: remoteConfig,
              command: "cp -R /tmp/code/* /www/wwwroot/${PROD_FILE_NAME}"
            )
            //更改文件所有者 更改文件夹权限 忽略掉更改.user.ini权限不足错误
            sshCommand(
              remote: remoteConfig,
              command: "chown -R www:www /www/wwwroot/${PROD_FILE_NAME} || true"
            )
            sshCommand(
              remote: remoteConfig,
              command: "chmod 755 -R /www/wwwroot/${PROD_FILE_NAME} || true"
            )
            //关闭维护模式
            sshCommand(
              remote: remoteConfig,
              command: "cd /www/wwwroot/${PROD_FILE_NAME} && php artisan up"
            )
            //删除暂存文件
            sshCommand(
              remote: remoteConfig,
              command: 'rm -rf /tmp/code.tar.gz'
            )
            sshCommand(
              remote: remoteConfig,
              command: 'rm -rf /tmp/code'
            )
            echo "部署成功"
          }
        }
      }
    }
  }
}
```

## 遇到的问题

- 由于coding登录用的账号是邮箱账号有`@`符，导致Coding仓库地址无法正确识别，所以先将Coding的账号和密码先进行URL转码，再存入设置中。
- 项目文件夹下的`.user.ini`防跨站文件无法修改权限，并且我们其实不需要修改它的权限，所以直接忽略掉这个错误

## 参考

- <https://blog.forecho.com/automatic-deployment.html>
- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [Jenkins 用户手册](https://www.jenkins.io/zh/doc/)
- [Coding 持续集成](https://coding.net/help/docs/ci/intro.html)

