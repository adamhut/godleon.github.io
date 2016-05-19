---
layout: post
title:  "[RHCE7] RH134 Chapter 12. Accessing Network Storage with SMB 學習筆記"
description: "此文章記錄學習 RHCE7 RH134 Chapter 12. Accessing Network Storage with SMB 留下的內容"
date: 2016-05-13 04:45:00
published: false
comments: true
categories: [linux]
tags: [Linux, RHCE, RH134]
---


12. Accessing Network Storage with SMB
======================================

## Manually mounting and unmouting SMB file systems

要裝 `cifs-utils` & `samba-client` 兩個套件

### 查詢

查詢 server 分享了什麼資源(匿名)：`smbclient -L //server0`

查詢 server 分享了什麼資源：`smbclient -L //server0 -U [UserName] -W [DomainName]` (`-U` 指定使用者，`-W` 指定網域)

### 掛載

匿名掛載：`sudo mount -t cifs -o guest //server0/public /mnt`

指定使用者掛載：`sudo mount -t cifs -o username=[UserName],password=[Password],workgroup=[DomainName] //server0/public /mnt` (若不加 `password` 會被要求手動輸入密碼)

## 透過 /etc/fstab 掛載

加入 `/etc/fstab` 中：`//server0/pubilic  /mnt  cifs  defaults,username=[UserName],password=[Password],workgroup=[DomainName] 0 0`

## 指定 credential 掛載

以 credential 的方式設定：`//server0/pubilic  /mnt  cifs  defaults,credentials=/root/smb.txt 0 0`

`/root/smb.txt` 的內容如下：(**only root access, chmod 600**)

```ini
username=[UserName]
password=[Password]
domain=[DomainName]
```

## Mounting SMB file systems with the automounter

除了 NFS 之外，SMB 也可以使用 automounter 來協助自動掛載，差別只有掛載參數上的不同，以下是設定範例：

- 本地目錄：`/bakerst/cases`

- 遠端目錄：`//serverX/cases`

1、安裝 `autofs` 套件

2、新增檔案 **/etc/auto.master.d/bakerst.autofs**，內容如下：

```bash
/bakerst  /etc/auto.bakerst
```

3、新增檔案 **/etc/auto.bakerst**，內容如下：

```bash
cases   -fstype=cifs,credentials=/secure/sherlock   ://serverX/cases
```

4、新增檔案 **/secure/sherlock**，內容如下：(**only root access**, permission **600**)

```ini
username=[UserName]
password=[Password]
domain=[DomainName]
```

5、啟動 autofs：`sudo systemctl enable autofs && sudo systemctl restart autofs`