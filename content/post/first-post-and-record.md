---
title: "First Post and Record"
date: 2019-06-08T19:53:15+08:00
lastmod: 2019-06-08T19:53:15+08:00
draft: false
keywords: []
description: "这是第一篇博客，是一个记录，也是一个开始"
tags: []
categories: ["default"]
author: "lc"
weight: 11

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: true 
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: true
  options: "这是第一篇博客，是一个记录，也是一个开始"

---

<!--more-->


按理说，首页应该放一个欢迎页。但是这篇博客作为第一篇，主要记录一下这几天来折腾这个hugo+travis+theme的过程。也记录一些其他的细节
首先是我的的需求分析：
1、 需要写博客。最好支持MarkDown，一定要支持公式
2、 博客最好有留言功能，有登陆控制。如果能定制就更好了。
3、 不要太复杂，并不想过多的设计前端。

一开始的时候，选择了flask，python的框架，轻量级，开发很快。也便于我用来做neo4j的展示。但是单纯想要博客，光MD编辑就够我折腾的了。
然后查到了Hugo，是Golang的框架，也没太管静态网站的事情，就直接上手了。这玩意有多种主题，还可以做CICD，用Travis集成，Github一更新，就直接自动构建部署。真是爽。
但是折腾下来，发现并不是很符合我的需求。
1、 主题很难修改。我连主页都控制不了！！
2、 功能需要开发，比如登陆控制和留言，这也太不方便了。
冷静思考，还是静态网页不符合我的需求。
回头一搜，我的需求其实WordPress都支持啊！我为啥非要用这Hugo，和自己过不去吗？？？？？
一番动手实践，确实WP是可以支持MD的，也支持公式。
但是这边的github Pages都已经配置了，还做了Travis的CICD，也不能放在这里浪费或者丢人啊。
好吧，那就两边同时更新吧。我会先在WP编辑好，然后再拷贝到这边更新。

最后记录一下这次折腾的一些收获：
1、 github免密登陆，就是现在自己的机器上ssh-gen一个新的密钥，然后传到github上，然后在git clone的时候使用ssh地址就可以了。
2、 Travis需要一个令牌。同时.travis文件注意用户名，我就是用户名和仓库名两边加了尖括号，导致deploy一直失败，还查不到原因。（一般低级错误都很难查到原因）
3、 Vim插入时间，有一个命令，可以映射到F5上，一查就知道。[strftime](https://blog.csdn.net/linwhwylb/article/details/6284286)
4、 github pages各种折腾，[这篇](https://segmentfault.com/a/1190000012975914)写的很好
5、 最后我的[博客](blog.tureornot.com)，从今天开始会每周写一篇，写成一个系列，从零开始NLP
