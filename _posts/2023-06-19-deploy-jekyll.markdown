---
layout:     post
title:      "配置jekyll本地环境"
subtitle:   "Deploy local environment for jekyll"
date:       2023-06-20 14:38:00
author:     "Lv Hui"
header-img: "img/bg/deploy-jekyll.png"
header-mask: 0.5
catalog: true
tags:
- jekyll
---

## 配置ruby

### 安装rbenv

使用homebrew安装rbenv,rbenv可以用来管理多个ruby版本。mac自带的ruby是2.6.10，而jekyll需要3.0.0版本以上，这里以安装3.0.6为例。

```zsh
# 安装 rbenv
brew install rbenv
```

为shell配置环境变量，我这里用的是zsh，所以在`~/.zshrc`中加入如下两行代码，并执行`source ~/.zshrc`使之生效

```zsh
export PATH="Users/lvhui/.rbenv/shims:$PATH
eval "$(rbenv init - zsh)"
```

### 安装ruby

使用rbenv安装ruby，切换到对应版本
```zsh
# 查看可用的ruby稳定版本
rbenv install --list
# 安装ruby
rbenv install 3.0.6
# 切换ruby版本
rbenv global 3.0.6
# 查看是否生效
ruby -v
```

## 安装jekyll

执行如下命令安装jekyll

```zsh
gem install --user-install bundler jekyll
```

为shell配置环境变量，在`~/.zshrc`中加入如下两行代码，并执行`source ~/.zshrc`使之生效

```zsh
export GEM_HOME="$(ruby -e 'puts Gem.user_dir')"
export PATH="$PATH:$GEM_HOME/bin"
```

### 启动本地服务

```zsh
# 创建新项目
jekyll new my-site
# 启动本地服务
bundle exec jekyll serve --trace
```

## 参考

- <https://jekyllrb.com/>
- <https://ruby-china.org/wiki/rbenv-guide>