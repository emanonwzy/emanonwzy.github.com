---
layout: post
title: "cofing blog"
description: ""
category: Others
tags: [其他]
---
{% include JB/setup %}

折腾了一晚上总算可以了。有以下几点要注意：
* 在windows上字符编码都换成utf8无BOM格式，显示就正常了。而且在本地和服务器上还不太一样。
* category只能为英文，因为这个会出现在URL中
* gem install jeykll时需要Devkit，这个和ruby要下相同的版本，不要一个32位一个64位，第一次
  本人就犯了这个错误，后来没办法又重装了一次
