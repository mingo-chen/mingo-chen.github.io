---
layout: post
title: 使用Fiddler对Http接口Mock处理
category: 工具
tag: [web, fiddler, http]
blog: true
---
### 简介 ###
在web项目中,会用到第三方的接口,那么在开发或者测试时,如何对第三方的接口进行mock呢?

{{ site.excerpt_separator }}

> mock是假装,模拟的意思

> 通过mock手段,可以让接口返回指定的数据,使业务测试流程得以顺利跑通

> 由于接口返回数据可控可改,测试就可以覆盖到所有分支/用例

最直接的做法是让接口提供者配合,一般难度极大

在此引入一神器--Fiddler网络代理+http请求断点的功能

Fiddler可以对本机上发出及收到的接口进行拦截(断点)

 - bpu指令: 对指定URL进行请求前拦截,修改用户提交到服务端的数据,即Request值
 - bpafter指令: 对指定URL进行请求后拦截,修改修改服务端返回的值,即Response值

### 实现步骤 ###
 - 开启本机网络监控并建立拦截规则

 ![对拦截对URL建立规则](/assets/images/fiddler/mock_http_response.png)
 
 拦截的URL可以支持正则表达式 

 - 对规则设置拦截动作

 对请求响应可以有多种处理方法,可以断点然后选择一个处理策略
 可以指定一个文件,此处直接指定模拟数据的文件
 这里我们想要接口返回期望的数据,所以选择准备好的mock数据文件

 ![设置接口返回值](/assets/images/fiddler/mock_http_repsonse2.png)
 
 - 在测试机开启网络代理

> export http_proxy=http://{ip}:{port}

 - 测试被拦截接口的返回值

 ![被代理并拦截的接口返回值](/assets/images/fiddler/after_set_http_proxy1.png)

 - 测试没被拦截接口的返回值

 ![被代理没拦截的接口返回值](/assets/images/fiddler/after_set_http_proxy.png)

 - 关闭测试机网络代理

> unset http_proxy

 - 测试被拦截接口的返回值

 ![关闭代理接口返回值](/assets/images/fiddler/before_set_http_proxy.png)

### 参考 ###
 * [web debugger fiddler 使用小结](http://www.cnblogs.com/forcertain/archive/2012/11/29/2795139.html)
 * [Fiddler Script 与 HTTP 断点调试](http://my.oschina.net/leejun2005/blog/399108)
 * [Fiddler 默认命令](http://blog.csdn.net/spring21st/article/details/5843495)
 * [关于 WEB/HTTP 调试利器 Fiddler 的一些技巧分享](http://my.oschina.net/leejun2005/blog/151103)
 * [前端开发利器—FIDDLER](http://www.cnblogs.com/yuzhongwusan/archive/2012/07/20/2601306.html?utm_source=tuicool&utm_medium=referral)
