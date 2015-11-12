---
layout: post
title: 安装 apk 遇到 INSTALL PARSE FAILED NO CERTIFICATES
guid: true
comments: true
categories:
  - android
tags: [adb]
---

前两天一位同学传来两个作品，通过 ADB 命令安装，结果显示 "Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES]"，显然两个文件都没有签名。

谷歌之，找到了解决办法，记录一下：

随便找一个签名文件，或者自己新建一个（若不会创建，请自行谷歌）。

1. 在终端下 cd 到目标 apk 目录
2. 输入 `jarsigner -verbose -keystore sample.keystore Test.apk sampleKeyStoreAliasName`，之后会要求输入 Alias 密码。
3. 输入 `jarsigner -verify Test.apk` 验证是否签名成功
4. 正常安装 apk

#### Credit
[Android: Solution "Install Parse Failed No Certificates"](https://dzone.com/articles/android-solution-install-parse-1)
