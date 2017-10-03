---
title: 如何在 Intellij 中设置集成 Jenkins 服务器连接
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
- Groovy
- Jenkins
tags:
- Groovy
- Jenkins
---

# 如何在 Intellij 中设置集成 Jenkins 服务器连接

在Intellij中可以很方便的设置Jenkins服务器，不用登录到浏览器中，在Intellij中即可浏览所有job，开发plugin，或利用现有plugin比如job-dsl轻松创建新Job，运行Job。

具体步骤如下：

1. 下载 Jenkins Control Plugin
![](http://i.imgur.com/YB0iPBv.png)

2. 重启 Intellij

4. 在 Intellij 中设置Jenkins 服务器，确保测试成功。

![](http://i.imgur.com/7cqQAoL.png)

注意：如果你用的是 jenkins 2, 并且启用了 CSRF(防止跨站点请求伪造），需要填 Crumb Data， 这个可以通过以下url获取:

http://<your jenkins server>/crumbIssuer/api/xml?tree=crumb

如果你不想启用 CSRF的话（一般不建议这样），到 Jenkins管理 -> Configure Global Security中, 取消勾选下图选项.

![](http://i.imgur.com/nrRVvVC.png)

3. Go to View->Tools&Windows->Jenkins，打开右侧边栏的 Jenkins 工作台。

4. 到你的jenkins 服务器浏览器页面上，手动创建一个简单的Job，回到Intellij, 刷新下，应该就可以看到这个job了。

![](http://i.imgur.com/xp0LtXO.png)

继续探索吧。
