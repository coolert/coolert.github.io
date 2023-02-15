---
layout:     post
title:      "CentOS上使用rclone挂载onedrive"
subtitle:   "Use rclone to mount the onedrive on CentOS"
date:       2023-02-15 17:00:00
author:     "Lv Hui"
header-img: "img/bg/rclone.png"
header-mask: 0.3
catalog: true
tags:
    - Linux
---


最近想着自用服务器平时也总是闲置，不如做个下载机，避免资源浪费。但是硬盘不大，毕竟云服务价格不低，所以准备挂载网盘弥补存储的不足。我这里用的是E5开发者的onedrive，一个子账号最大有5T的空间，总共能有25T，妥妥够用了。

# 安装rclone

## 脚本下载及安装

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

不知道是不是因为我用的是国内的服务器的原因，脚本安装不成功，毕竟是凉心云，所有只能选择编译安装。

## 编译安装

```bash
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64
sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone
sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb
```

更多的安装可以查看官方文档 ：[https://rclone.org/install/](https://rclone.org/install/)

## 获取授权信息

由于服务器上没有图形界面，所以没办法用浏览器进行授权，所以需要借助其他的电脑进行授权，这里以windows为例。更多可查看官网：[https://rclone.org/remote_setup/](https://rclone.org/remote_setup/)

### 下载文件

在官网下载符合自己电脑版本的文件，下载地址：h[ttps://rclone.org/downloads/](https://rclone.org/downloads/)

我下载的是window64位文件，解压后得到文件夹 `rclone-v1.61.1-windows-amd64`

打开cmd或者powershell进入到该文件夹执行

```bash
rclone.exe authorize "onedrive"
```

之后浏览器会弹出一个授权登录页，输入账号与密码，然后点击同意授权即可。

命令窗口会显示如下信息，一串授权信息的json

```bash
#复制整个json，即 ---> 与 <--- 之间的所有内容，保存下来之后会用到
Paste the following into your remote machine --->
{"access_token":"xxxx","token_type":"xxxx","refresh_token":"xxxx","expiry":"xxxx"} 
<---End paste
```

## 配置rclone

执行配置命令

```bash
rclone config
```

具体的流程

```bash
[root@VM-4-17-centos ~]# rclone config
Current remotes:
Name                 Type
====                 ====

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n #选择n新建

Enter name for new remote.
name> mary #为挂载的网盘取一个名称

Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ (fichier)
 2 / Akamai NetStorage
   \ (netstorage)
 3 / Alias for an existing remote
   \ (alias)
 4 / Amazon Drive
   \ (amazon cloud drive)
 5 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, China Mobile, Cloudflare, ArvanCloud, DigitalOcean, Dreamhost, Huawei OBS, IBM COS, IDrive e2, IONOS Cloud, Liara, Lyve Cloud, Minio, Netease, RackCorp, Scaleway, SeaweedFS, StackPath, Storj, Tencent COS, Qiniu and Wasabi
   \ (s3)
 6 / Backblaze B2
   \ (b2)
 7 / Better checksums for other remotes
   \ (hasher)
 8 / Box
   \ (box)
 9 / Cache a remote
   \ (cache)
10 / Citrix Sharefile
   \ (sharefile)
11 / Combine several remotes into one
   \ (combine)
12 / Compress a remote
   \ (compress)
13 / Dropbox
   \ (dropbox)
14 / Encrypt/Decrypt a remote
   \ (crypt)
15 / Enterprise File Fabric
   \ (filefabric)
16 / FTP
   \ (ftp)
17 / Google Cloud Storage (this is not Google Drive)
   \ (google cloud storage)
18 / Google Drive
   \ (drive)
19 / Google Photos
   \ (google photos)
20 / HTTP
   \ (http)
21 / Hadoop distributed file system
   \ (hdfs)
22 / HiDrive
   \ (hidrive)
23 / In memory object storage system.
   \ (memory)
24 / Internet Archive
   \ (internetarchive)
25 / Jottacloud
   \ (jottacloud)
26 / Koofr, Digi Storage and other Koofr-compatible storage providers
   \ (koofr)
27 / Local Disk
   \ (local)
28 / Mail.ru Cloud
   \ (mailru)
29 / Mega
   \ (mega)
30 / Microsoft Azure Blob Storage
   \ (azureblob)
31 / Microsoft OneDrive
   \ (onedrive)
32 / OpenDrive
   \ (opendrive)
33 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ (swift)
34 / Oracle Cloud Infrastructure Object Storage
   \ (oracleobjectstorage)
35 / Pcloud
   \ (pcloud)
36 / Put.io
   \ (putio)
37 / QingCloud Object Storage
   \ (qingstor)
38 / SMB / CIFS
   \ (smb)
39 / SSH/SFTP
   \ (sftp)
40 / Sia Decentralized Cloud
   \ (sia)
41 / Storj Decentralized Cloud Storage
   \ (storj)
42 / Sugarsync
   \ (sugarsync)
43 / Transparently chunk/split large files
   \ (chunker)
44 / Union merges the contents of several upstream fs
   \ (union)
45 / Uptobox
   \ (uptobox)
46 / WebDAV
   \ (webdav)
47 / Yandex Disk
   \ (yandex)
48 / Zoho
   \ (zoho)
49 / premiumize.me
   \ (premiumizeme)
50 / seafile
   \ (seafile)
Storage> 31 #选择31挂载onedrive

Option client_id.
OAuth Client Id.
Leave blank normally.
Enter a value. Press Enter to leave empty.
client_id> #直接回车

Option client_secret.
OAuth Client Secret.
Leave blank normally.
Enter a value. Press Enter to leave empty.
client_secret> #直接回车

Option region.
Choose national cloud region for OneDrive.
Choose a number from below, or type in your own string value.
Press Enter for the default (global).
 1 / Microsoft Cloud Global
   \ (global)
 2 / Microsoft Cloud for US Government
   \ (us)
 3 / Microsoft Cloud Germany
   \ (de)
 4 / Azure and Office 365 operated by Vnet Group in China
   \ (cn)
region> 1 #地区之前选1全球

Edit advanced config?
y) Yes
n) No (default)
y/n> #直接回车

Use web browser to automatically authenticate rclone with remote?
 * Say Y if the machine running rclone has a web browser you can use
 * Say N if running rclone on a (remote) machine without web browser access
If not sure try Y. If Y failed, try N.

y) Yes (default)
n) No
y/n> n #这里选n由于没有浏览器，所有需要借助其他电脑来进行授权信息的获取

Option config_token.
For this to work, you will need rclone available on a machine that has
a web browser available.
For more help and alternate methods see: https://rclone.org/remote_setup/
Execute the following on the machine with the web browser (same rclone
version recommended):
	rclone authorize "onedrive"
Then paste the result.
Enter a value.
config_token> #这里填入我们之前保存的授权信息,整个粘贴然后回车

Option config_type.
Type of connection
Choose a number from below, or type in an existing string value.
Press Enter for the default (onedrive).
 1 / OneDrive Personal or Business
   \ (onedrive)
 2 / Root Sharepoint site
   \ (sharepoint)
   / Sharepoint site name or URL
 3 | E.g. mysite or https://contoso.sharepoint.com/sites/mysite
   \ (url)
 4 / Search for a Sharepoint site
   \ (search)
 5 / Type in driveID (advanced)
   \ (driveid)
 6 / Type in SiteID (advanced)
   \ (siteid)
   / Sharepoint server-relative path (advanced)
 7 | E.g. /teams/hr
   \ (path)
config_type> 1 #选1

Option config_driveid.
Select drive you want to use
Choose a number from below, or type in your own string value.
Press Enter for the default (b!rHXWeebpwkuiuJyInal9KK2gUZ7iAtBIlvntEswCDQ4ZyrruoCV5SJi9PcL-NtJ5).
 1 / OneDrive (business)
   \ (b!rHXWeebpwkuiuJyInal9KK2gUZ7iAtBIlvntEswCDQ4ZyrruoCV5SJi9PcL-NtJ5)
config_driveid> 1 #选1

Drive OK?

Found drive "root" of type "business"
URL: https://****-my.sharepoint.com/personal/mary_****_onmicrosoft_com/Documents

y) Yes (default)
n) No
y/n> #直接回车

Configuration complete.
Options:
- type: onedrive
- token: {"access_token":"****"}
- drive_type: business
Keep this "mary" remote?
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> #直接回车

Current remotes:

Name                 Type
====                 ====
mary                 onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q #选择q退出
```

# 挂载网盘

## 安装fuse

挂载时需要用到fuse

```bash
yum install fuse
```

## 挂载

新建需要挂载的文件夹，我准备挂载到home下的一个新文件夹，所以执行

```bash
mkdir /home/onedrive
```

执行挂载

```bash
rclone mount mary:/ /home/onedrive --copy-links --allow-other --allow-non-empty --umask 000 --daemon
```

查看是否挂载成功 

```bash
[root@VM-4-17-centos ~]# df -h
文件系统        容量  已用  可用 已用% 挂载点
devtmpfs        1.9G     0  1.9G    0% /dev
tmpfs           1.9G   40K  1.9G    1% /dev/shm
tmpfs           1.9G  740K  1.9G    1% /run
tmpfs           1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/vda1        79G   25G   52G   33% /
tmpfs           379M     0  379M    0% /run/user/0
tmpfs           379M     0  379M    0% /run/user/1003
overlay          79G   25G   52G   33% /var/lib/docker/overlay2/a798202d5aa6d944c0ed3c994e22b09b375de8b2bf58c52122d1d2462ca42ed6/merged
mary:           5.0T   30G  5.0T    1% /home/onedrive
```

在最后一行可以看到已经挂载成功了

## 卸载

```bash
fusermount -qzu /home/onedrive/
```