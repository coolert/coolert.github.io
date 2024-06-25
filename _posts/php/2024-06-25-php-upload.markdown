---
layout:     post
title:      "一些影响 PHP 上传文件大小的配置"
subtitle:   "Some configurations that affect the size of PHP uploaded files"
date:       2024-06-25 14:22:00
author:     "Lv Hui"
header-img: "/lsky.coolerk.com/i/2024/06/21/667546a311aca.png"
header-mask: 0.6
catalog: true
tags:
    - PHP
---

## PHP 配置

1. **upload_max_filesize**
    
    该配置限制单个上传文件的大小，默认值通常为2M

2. **post_max_size** 

    该配置限制POST数据的最大大小，包括表单数据和文件上传数据，该值不能小于 upload_max_filesize ，默认值通常为8M

3. **memory_limit**

    该配置限制PHP可使用的最大内存，默认值通常为128M

    内存限制可以的在代码中进行动态调整，使用`ini_set()`函数进行配置

    ```php
    ini_set('memory_limit', '512M');
    ``` 

## Nginx 配置

Nginx 默认文件上传大小限制为1M，修改 `client_max_body_size` 项就可以更改限制的值了


