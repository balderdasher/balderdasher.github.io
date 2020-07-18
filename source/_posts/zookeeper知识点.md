---
title: 解决idea类报红找不到的问题
date: 2017-10-11 12:13:06
tags: idea
categories: 奇门遁甲
permalink: fix-idea-show-red-on-codes
---

国庆假期回来之后打开电脑，ide打开之后发现之前一切正常的程序代码出问题了：具体表现为某个或某几个类引用报红，提示：`Cannot resolve symbol 'SomeClass'`，但是在出现问题的类中执行`main()`方法确实没有问题的，就像下图所展示的一样：

![](https://i.imgur.com/nifLGi5.png)
<!-- more -->
刚开始我以为是ide的bug，网上搜了一下，原来是缓存的问题：解决办法如下

![](https://i.imgur.com/PMHXqrj.png)

![](https://i.imgur.com/aL8zqMT.png)