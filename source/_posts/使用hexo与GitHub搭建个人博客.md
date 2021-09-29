---
title: 使用hexo与GitHub搭建个人博客
date: 2021-09-29 15:54:49
tags: hexo
categories: 奇门遁甲
permalink: build-blog-with-hexo-vs-github
---

## 参考资料

* https://blog.csdn.net/gdutxiaoxu/article/details/53576018
* https://www.zhihu.com/question/21193762/answer/134278844
* https://www.jianshu.com/p/f054333ac9e6

## 注意事项

### 一.日常改动流程

在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理：
1. 依次执行`git add .`、`git commit -m "..."`、`git push origin hexo`指令将改动推送到GitHub（此时当前分支应为hexo）
2. 然后才执行`hexo g -d`发布网站到master分支上

### 一.换了电脑之后怎么办

1. 使用`git clone` 命令拉取GitHub仓库（默认分支为hexo）
2. 在本地仓库文件夹下通过Git bash依次执行下列指令：`npm install hexo`、`npm install`、`npm install hexo-deployer-git`（记得，不需要`hexo init`这条指令）
