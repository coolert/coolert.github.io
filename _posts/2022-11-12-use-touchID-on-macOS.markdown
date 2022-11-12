---
layout:     post
title:      "在macOS上使用TouchID验证sudo"
subtitle:   "Use TouchID to Authenticate sudo on macOS"
date:       2022-11-12 23:51:00
author:     "Lv Hui"
header-img: "img/bg/macos-ventura-surf.jpg"
header-mask: 0.3
catalog: true
tags:
    - macOS
---


每次输入`sudo`命令时都需要输入密码验证，使用TouchID验证可以方便许多。如果你有apple watch 还可以在apple watch 上双击验证。

使用文本编辑器打开 `/etc/pam.d/sudo`文件，或者直接在vim中打开

```bash
sudo vim /etc/pam.d/sudo
```

在 `auth sufficient pam_smartcard.so`的下一行加入`auth sufficient pam_tid.so` 并保存

```bash
# sudo: auth account password session
auth       sufficient     pam_smartcard.so
auth       sufficient     pam_tid.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```

如果用vim打开的时候忘记了加`sudo`命令，保存时会提示为只读文件，使用`:w !sudo tee %` 保存，这样就可以生效了

但是每次系统更新的时候，该文件会被重置，所以需要重新修改该文件

### 参考

- ****[Use TouchID to Authenticate sudo on macOS](https://it.digitaino.com/use-touchid-to-authenticate-sudo-on-macos/)****