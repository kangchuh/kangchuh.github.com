---
layout: post
title: "Xcode Tips"
date: 2014-10-21 22:58:16 +0800
comments: true
categories: iOS Xcode
---
<br/>

在使用Xcode的时候，经常会遇到各种“莫名其妙”的‘BUG’。本来写完一段完美的代码，想来测试一下自己的成果，结果，**run**，Xcode给你各种莫名的xxx信息，这会你要恼羞成怒了。

###一、问题

#####1.Could not inspect the application package.

在 Xcode 6 上出现一次正常，一次弹出 Could not inspect the application package 错误问题，原因可能是：
* 项目 **Product Name** 包含非拉丁文（即：包含中文），所以只需要把你的 Product Name 修改一下就 OK 可以；
* 项目包含 *resource / resources* 文件夹，可能与系统内部冲突，所以把名称改一下就可以；
* 项目在运行中，没有 stop 就直接 run 可能也会导致此问题，这个问题，养成好习惯，先 stop 在 run。
<br/>
参考：[Xcode: Could not inspect the application package](http://bit.ly/ZCNvOq)