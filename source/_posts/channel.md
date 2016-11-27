---
title: Android使用脚本多渠道打包
date: 2016-09-20 19:52:42
categories: 工具
keywords: 多渠道, 脚本,打包,Android studio打包
tags: [Android多渠道,打包]
---
本文参考自https://github.com/pengjianbo/MutiChannelPackup
的Python脚本，对脚本作了扩展:
- 支持命令行参数，支持多flavor渠道打包
- 路径优化，支持配置多个flavor，多个flavor对应相同或不同的渠道
- 简化调用方式
- 支持渠道号与文件命名不一样，渠道号有对应的文件名字
<!--more-->
对不会Python的同学使用更加亲和。

早期使用Eclipse开发Android的同学想必对多渠道打包的痛苦还记忆犹新, 好在使用Android Studio后这一切变得简单起来(虽然还是很慢)。如果仅仅是几个渠道，使用gradle配置已经非常方便了。
设想这样一种场景：应用要在各大市场上线(20个起码)，还有特别定制的(对企业个别页面和资源定制)也要在各大市场上线(20个起码)，这样30到40个包打出来30分钟左右(我8G内存的PC),给点耐心估计也能忍，如果遇上司机或者发现bug重新打包，还能忍（有摔电脑的冲动）?

本文思路为META-INF渠道识别,你可以接着往下看，或者更详细请查看美团team博客[点击查看](http://tech.meituan.com/mt-apk-packaging.html)

至于为什么选用Python脚本，本人试了下，感觉Python最方便，而且作为一个Python爱好者,知道了META-INF渠道识别的原理，需要作的知识文件操作而已，很修改原脚本。

#### META-INF渠道识别机制
我们普通apk打完包解压后是这样的结构：
![apkmeta](http://odsvc7v7q.bkt.clouddn.com/apkmetainfo.png)
主要在META-INF里面作文章，在META-INF里面添加文件不需要重新签名，步骤大概这样:
- 把渠道号以文件名的方式(带前缀方便读取)通过脚本写入到这个文件夹
- 在android代码中读取带约定前缀的文件的渠道号，设置渠道号

打包完后META-INF文件夹这样：
![META-INF](http://odsvc7v7q.bkt.clouddn.com/channelmeta.png)
10004就是后面要设置的渠道号

#### 使用方式
##### 调用
先说调用方式，简单配置后(只需要generator.py,info.conf两个文件)，AS打出我们要设置渠道的所有Flavor包，命令行调用：
> python generator.py [flavor] [flavor]..

![cmdflavor](http://odsvc7v7q.bkt.clouddn.com/cmdbuild.png)
这里就在apk_normal文件夹下生成了我的17个渠道包
flavor为可选，不填写生成默认的，要同时生成多个flavor的渠道包可以这样：

> python generator.py normal dianxin

##### 配置(info.conf)
我的配置：
> [Meta]

>apk.path = ./aa.apk

>dianxin_path = ../app/build/outputs/apk/dianxin_V2.3.1.apk

>normal_path = ../app/build/outputs/apk/normal_V2.3.1.apk

>channel_prefix = channel-

>[dianxin]

>google=20001

>hiapk=20002

>[normal]

>hiapk=10001

>360cn=10002

我这里是电信和normal两个flavor 程序表现有些不一样，分别打渠道包，我的脚本在根目录下建了个文件夹，放在里面，可以随便放，修改路径即可，有问题可以在下方留言。

python代码和我的配置 [github地址](https://github.com/zyongjun/python-cookie/tree/master/androidbuild)，如果觉得有帮助就给个star吧

以上，祝编程愉快 :)<br/>
感谢阅读此文章，如果此文章对你有用或者你有任何疑问和意见，请在下方留言或者在[github](https://github.com/zyongjun)上fork我,你的鼓励对我有非凡的意义。
