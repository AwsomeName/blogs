---
title: "Make_github_pages"
date: 2019-06-11T16:48:53+08:00
lastmod: 2019-06-11T16:48:53+08:00
draft: false
keywords: []
description: ""
tags: ["blog"]
categories: ["blog"]
author: "lc"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
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
  enable: false
  options: ""

---

之前的胡言乱语中，说明了选择Hugo的理由。使用后的体验也是，做一个静态的网站确实很合适，上手很快。缺点就是静态网页本身的缺点，想搞成动态的不容易。这里就记录一下如何利用GitHub搭建免费的个人静态博客。
<!--more-->
### 准备
在GitHub搭建并不难，只要在GitHub中建立<username>.github.io的项目，然后把网站相关的文件提交到这个项目中，就可以在https://<username>.github.io访问到这个网页。

#### 安装Hugo
    sudo apt install hugo
	hugo version
	
#### 生成网站
	hugo new site blog
	cd blog
	git init

观察blog下的目录：
-	`config.toml`是配置文件，在里面可以定义博客地址、构建配置、标题、导航栏等等。
-	`themes`是主题目录。这是一个很重要的目录，一个网站最终展现出什么样子，就是这个主题决定的。所以如果需要定制这个网站，就需要制定这个主题。可以去[themes.gohugo.io](https://themes.gohugo.io)下载他人分享的主题
-	`content`是博客文章的目录

#### 安装主题
可以将下载的主题直接保存到themes目录中，然后在config.toml中配置`theme = "even"`。这里推荐even主题。这里推荐以下方法：
```
git submodule add https://github.com/olOwOlo/hugo-theme-even.git themes/even
```

#### 第一篇文章
```
hugo new post/my-first-post.md
```
在文章的顶部可以设置一些meta信息，注意draft选项，需要设置成false，才会在最终的页面中显示。其中weight选项决定了页面中文章的排序。

#### 预览
可以使用Hugo生成静态内容并在本地启动server。
```
hugo server -D
```
其中D的参数是可以将draft为true的也显示。然后就可以在https://localhost:1313看到。任何改变也可以随时显现，Hogo server会检测文件变化，自动刷新浏览器。

#### 部署到GitHub Pages
1.	首先把源码提交到GitHub的一个repo
```
git add .
git commit -m "initial all files"
git remote add origin https://github.com/<username>/blog
git push origin master
```
2.	在GitHub上创建<username>.github.io，同时在config.toml的baseURL要设置成https://<username>.github.io
3.	生成GitHub Access Token，至少要有public_repo权限。
4.	配置Travis。去其官网注册关联GitHub账号，然后同步账号，并激活blog仓库。进入blog的设置页面，并选择自动部署触发条件，并把刚刚生成的GitHub Access Token添加到环境变量里。
5.	在blog中添加.travis.yml
```
sudo: false
language: go
git:
    depth: 1
install: go get -v github.com/gohugoio/hugo
script:
    ‐ hugo
deploy:
    provider: pages
    skip_cleanup: true
    github_token: $GITHUB_TOKEN
    on:
        branch: master
    local_dir: public
    repo: <username>/<username>.github.io
    fqdn: <custom-domain-if-needed>
    target_branch: master
    email: <github-email>
    name: <github-username>
```
注意
-	实际的name没有两边的尖括号。
-	默认情况下，travis会自动下载git submodules
-	github_token:$GITHUB_TOKEN要和travis设置的环境变量名一致。


### 参考
[使用 Hugo 搭建博客](https://segmentfault.com/a/1190000012975914)



