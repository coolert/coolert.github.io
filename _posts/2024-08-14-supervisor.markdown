---
layout:     post
title:      "Supervisor 安装与配置"
subtitle:   "Supervisor Installation and Configuration"
date:       2024-08-14 11:30:00
author:     "Lv Hui"
header-img: "img/post-bg-os-metro.jpg"
header-mask: 0.6
catalog: true
tags:
    - linux
---

## 安装

```shell
sudo apt update
sudo apt install supervisor
```

## 配置

supervisor 配置目录一般位于 /etc/supervisor/conf.d，我们新建一个配置文件worker.conf

```apacheconf
[program:worker]
process_name=%(program_name)s_%(process_num)02d
command= cd /path/to && ls
autostart=true
autorestart=true
user=username
numprocs=1
redirect_stderr=true
stdout_logfile=/path/to/worker.log
stderr_logfile=/path/to/worker_err.log
directory=/path/to/your/project
```

配置选项说明

- program:worker: 定义了进程的名称。
- process_name=%(program_name)s_%(process_num)02d: 用于命名每个队列进程，%(process_num)02d 会给进程编号，如 worker_00，worker_01。
- command: 指定要执行的命令。
- autostart=true: 表示 Supervisor 启动时自动启动该进程。
- autorestart=true: 如果进程意外退出，Supervisor 将自动重启它。
- user=username: 指定运行进程的用户。
- numprocs=1: 启动多少个进程，这里设置为1个进程.
- redirect_stderr=true: 将标准错误输出重定向到日志文件。
- stdout_logfile: 指定日志文件路径，用于记录队列工作的输出。
- stderr_logfile: 指定错误日志文件路径。
- directory=/path/to/your/project: 指定命令执行的目录。

### 加载配置

```shell
sudo supervisorctl reread
sudo supervisorctl update
```

### 管理进程

```shell
# 启动
sudo supervisorctl start worker:*
# 停止
sudo supervisorctl stop worker:*
# 重启
sudo supervisorctl restart worker:*
# 查看状态
sudo supervisorctl status
```