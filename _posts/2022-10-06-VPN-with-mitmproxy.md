---
layout: post
title: "同时使用VPN代理以及mitmproxy抓包"
author: "Toseic"
tags: crawl trick web
comments: true
excerpt_separator: <!--more-->
audio: ""

---

## mitmdump

如果我们想在开启了`VPN`的情况下如何实现抓取通过`VPN`返回的流量包呢，

首先设我们`VPN`的代理端口为`P1`，我们可以通过执行：

```bash
mitmdump --mode upstream:http://127.0.0.1:${P1}/ -s ./${path)/script.py
```

访问来自`VPN`代理端口的流量。

## charles

`charles`也可以使用类似的方法抓包，通过设置`上流流量服务器`即可。