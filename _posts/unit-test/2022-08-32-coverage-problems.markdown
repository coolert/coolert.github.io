---
layout:     post
title:      "Homestead环境下phpunit生成代码覆盖率报告的一些问题"
subtitle:   "Some problems using phpunit generate coverage report in homestead"
date:       2022-08-31 15:56:37
author:     "Lv Hui"
header-img: "img/unit-test/bg.jpg"
header-mask: 0.3
catalog: true
tags:
    - phpunit
    - 单元测试
---

## 环境

- php 7.3
- phpunit 9.2.15

## XML配置文件

如果没有使用`–configuration`phpunit会优先使用`phpuint.xml`配置，如果不存在便会使用`phpunit.xml.dist`配置。也就是`phpunit.xml`的优先级要高于`phpunit.xml.dist`，所以可以将`phpunit.xml.dist`文件加入到版本控制中，将`phpuni.xml`加入到`.gitignore`中。如果有人想要运行修改过的测试文件的话，可以将`phpunit.xml.dist`文件中的内容复制到`phpunit.xml`中进行修改并运行测试，这样既可以保证测试能正常运行，又不会影响版本控制中的xml配置。

`phpunit.xml.dist` 配置内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" backupGlobals="false" backupStaticAttributes="false" bootstrap="vendor/autoload.php" colors="true" convertErrorsToExceptions="true" convertNoticesToExceptions="true" convertWarningsToExceptions="true" processIsolation="false" stopOnFailure="false" xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/9.3/phpunit.xsd">
  <coverage>
    <include>
      <directory suffix=".php">src/</directory>
    </include>
    <report>
      <html outputDirectory="html-coverage"/>
    </report>
  </coverage>
  <testsuites>
    <testsuite name="Application Test Suite">
      <directory>./tests/</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

`<coverage>`标签内包含覆盖率报告的具体设置，`<include>`标签内为要生成覆盖率报告的具体文件设置，`<report>`标签则用于生成报告的类型与保存位置等设置。上面的配置表示为`src`文件夹及其子文件夹下所有后缀为`.php`的文件生成覆盖率报告，生成`html`格式的报告，保存在`html_coverage`文件夹下。

不同版本的phpunit配置文件会有差异，可以看文档自行配置 [https://phpunit.de/documentation.html](https://phpunit.de/documentation.html)

## 显示 No code coverage driver available

phpunit使用的[php-code-coverage](https://github.com/sebastianbergmann/php-code-coverage)组件，使用了PHP 的 [Xdebug](https://xdebug.org/)或 [PCOV](https://github.com/krakjoe/pcov)扩展或 [PHPDBG](https://www.php.net/manual/en/book.phpdbg.php)所提供的代码覆盖率功能。可能是没有安装扩展，也可能是没能正确配置导致的，这里只说没有正确配置Xdebug导致的问题。

### 查看是否已加载xdebug

输入`php -m` 查询，看到并没有加载xdebug

![check modules](/img/unit-test/check-modules.png)

### 为php-cli加载xdebug

```bash
# 找到xdebug模块配置，本地用的php-cli是7.3版本
cd /etc/php/7.3/mode-available
# 查看是否有xdebug配置文件，如果没有需要自行安装xdebug
ls | grep xdebug
# 为xdebug配置创建软链接，提示权限不足记得前面加sudo，如果修改的是php-fpm的配置需要重启服务才能生效
ln -s /etc/php/7.3/mods-available/xdebug.ini /etc/php/7.3/cli/conf.d/20-xdebug.ini
# 配置php.ini
cd /etc/php/7.3/cli
vim php.ini
```

`php.ini` 最后一行加上 `xdebug.mode =  coverage`

![change setting](/img/unit-test/change-setting.png)