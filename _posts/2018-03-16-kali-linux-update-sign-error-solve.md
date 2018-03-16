---
layout: post
title: kali-linux签名更新
categories: Web安全
description: kali linux update操作时出现签名无效解决方法
keywords: kali, 签名无效
---

### 更新kali linux时出现

``W: 校验数字签名时出错。此仓库未被更新，所以仍然使用此前的索引文件。GPG 错误：http://mirrors.aliyun.com/kali kali-rolling InRelease: 下列签名无效： EXPKEYSIG ED444FF07D8D0BF6``

### 使用如下命令更新签名

``wget -q -O - <https://archive.kali.org/archive-key.asc>  | apt-key add``