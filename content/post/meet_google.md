---
title: "Meet_google"
date: 2019-06-09T17:16:28+08:00
lastmod: 2019-06-09T17:16:28+08:00
draft: false
keywords: ["aws","ss"]
description: ""
tags: ["aws","ss"]
categories: ["aws","ss"]
author: "lc"

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
[原文](http://blog.tureornot.com/2019/06/08/梯子启动大法/)
expressvpn 挂了，没办法
sslocal -c /etc/shadowsocks/config.json 这条命令是在自己本机启动代理的。下面记录一下完整的流程：
<!--more-->
## 服务器端
***
1.	首先启动AWS的EC2服务，这一步就不多说了，需要注意的是，安全组，要开启TCP的全部，不然会链接不上。
2.	登陆到EC2，安装shadowsocks服务器版：
	sudo apt update
	sudo apt install python-pip
	sudo pip install shadowsocks
3.	启动服务，并检查端口：
	sudo ssserver -p 8989 -k password -m aes-256-cfb -d start
4.	检查端口是否启动：
	netstat -tln
以上服务器端就完成了，下面是自己的机器端

## 客户端
***
1.	安装shadowsocks
	apt-get install shadowsocks
2.	配置文件
	sudo vim /etc/shadowsocks/config.json
	
	{
		"Server":"aws ec2 ip"
		"Server_port":
		"local_port": 1080
		"password":"u passwd"
		"timeout": 600
		"method": "aes-256-cfb"
	}
3.	启动
	nohup /user/bin/sslocal -c /etc/shadowsocks/config.json

## 启动遇到的问题和解决方案
***
1.	错误如下：
	AttributeError: /usr/local/ssl/lib/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
	shadowsocks start failed
2.	解决如下：
	1.	vim打开opsnssl.py（路径自行寻找）。
		vim /usr/local/lib/python3.5/dist-packages/shadowsocks/crypto/openssl.py
	2.	修改libcrypto.EVP_CIPHER_CTX_cleanup.argtypes
		:%s/cleanup/reset/
		:x
	>这是两条vim命令，替换文中的cleanup为reset，并保存退出。
	3.	重启shadowsocks.具体原因，见[参考文献](https://kionf.com/2016/12/15/errornote-ss/)

## 最后一步，浏览器插件
***
这里使用的是chrome浏览器，选择的是Switchymega插件。这里的安装并不难，墙内也很容易搜到，就不详细讲了。大体流程就是下好插件，然后浏览器设置打开开发者模式，安装插件，本地安装。安装成功后，将代理地址填为本地client即可。

参考文献：
[AWS安装ss server](http://www.bioinfo-scrounger.com/archives/490)
[ubuntu安装ss client](https://blog.csdn.net/eliuxiaoming1/article/details/85772050)
[ss 启动bug](https://kionf.com/2016/12/15/errornote-ss/)
[ss Chrome插件SwitchyOmega](http://www.wangchao.info/1316.html)

