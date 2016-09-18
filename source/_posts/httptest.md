---
title: 网络测试服务httpbin
date: 2016-09-19 00:11:23
categories: 工具
keywords: 网络测试, httpbin
tags:  [网络测试, 工具]
---

设想这样的场景：开发或者学习过程中，我们主要针对网络框架的搭建，没有特定的业务和接口，或者说手头上有些接口，数据太复杂。我们只需要方便专注于测试框架的可行性,有什么办法？
<!--more-->
针对前面的场景，我们一般有这样的解决办法：
- 定义假数据，手动返回（比较繁琐，可能引入bug）
- 使用Fiddler2或者charles这样的抓包工具，拦截并自定义返回(对不熟悉的同学不容易上手，移植性差，换台电脑需要配置返回规则)

前面两种方法都没什么问题，不好的一点是数据出现问题修改代码可能性比较大，另外比较繁琐。这里介绍第三种方式，使用httpbin。

#### [httpbin](http://httpbin.org/)介绍
httpbin是一个网络请求响应测试网站，支持Get,Post,Put,Delete等方式，支持调试Header数据(返回对应的json),支持返回svg,png,webp,jpeg图片。其他不一一列举，能满足各种网络测试场景，这里主要以这几种为例。

#### Post
![图片](images/post.png)

#### Header调试

![img](images/httpbinheaders.png)

#### 获取图片

![img](images/httpbin_img.png)

是不是非常强大，在测试网络框架或者爬虫的网络测试都非常方便，用起来吧。

以上，祝编程愉快 :)<br/>
感谢阅读此文章，如果此文章对你有用或者你有任何疑问和意见，请在下方留言或者在[github](https://github.com/zyongjun)上fork我,你的鼓励对我有非凡的意义。
