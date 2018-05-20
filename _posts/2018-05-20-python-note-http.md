---
title        : Python学习(2) - Http
author       : ahcming
description  :
date         : 2018-05-20 15:15
layout       : post
tag          : ["work", "note", "python"]
blog         : true
---

## Http概述

- HTTP是一个客户端和服务器端请求和应答的标准
- 客户端是指发起请求的一方, 不仅限手机,浏览器等设置, 可以是服务器A请求服务器B
- 服务端是指收到请求并处理请求的一方

### HttpRequest

请求包含请求头(Header), 请求体(Body)

- Header包含的重要信息
    * Content-Type: 页面内容的格式, application/html, application/json, application/protobuf等
    * Cookie: 网站的一些信息存在浏览器本地, 一般是用户行为数据, 非敏感数据
    * Host: 页面的域名, 防跨域
    * Referer: 页面的跳转来源, 上一页面, 一般用来防盗链
    * User-Agent: 用户的客户端信息, 用于页面兼容

- Body
    * 提交到服务器的数据

- Method
    * GET

    > 获取/查询数据

    * POST

    > 提交数据

    * PUT

    > 更新数据, 不常用, 大家常用POST

    * DELETE

    > 删除数据, 不常用, 大家常用POST


### HttpResponse

- Http Code

| code | 含义        | 说明                                                                    |
|:----:|:-----------:|:-----------------------------------------------------------------------:|
| 200  | 正常        | 无                                                                      |
| 204  | 无内容      | 常用来浏览器跨域标识                                                    |
| 302  | 重定义      | 旧的接口已经失效, 返回新的接口地址, 让客户端重新请求                    |
| 404  | 地址不存在  | 通常接口废弃, 图片文章删除会出现                                        |
| 413  | 请求体太长  | 现代浏览器一般没有限制GET请求体大小, 出现此情况,可调整nginx/jetty的配置 |
| 500  | 服务器异常  | 查看服务器日志                                                          |
| 502  | Bad Gateway | 查看服务器的配置                                                        |

## Python3+通过urllib库实现常用的Http请求


### Get请求
此处以获取知乎推荐问题接口为例

> https://www.zhihu.com/question/33277508

```python
import urllib.request

url = 'https://www.zhihu.com/question/33277508'

# 构建请求体, 一般Request接收url, headers, data3个参数, 具体用法见下面的样例
hot_req = urllib.request.Request(url)

# 发送请求
hot_resp = urllib.request.urlopen(hot_req).read()

# 对返回的数据进行解码
hot_data = hot_resp.decode('utf-8')

print("data: %s" % hot_data)
```

可用以下等价命令来查看请求细节

> curl -v https://www.zhihu.com/question/33277508

### Post请求

一般是用来提交数据, 比如登录, 发评论, 点赞等

样例代码如下:

```python
import urllib.request
import urllib.parse

url = 'http://www.iqianyue.com/mypost'

# 此header设置对请求没有什么用, 只是为了展示而写
head = {
    'User-Agent': 'AppleWebKit/537.36 (KHTML, like Gecko)',
    'Referer': 'https://ahcming.github.io',
    'Connection': 'keep-alive'
}

# 要提交到服务器的数据
post_origin_data = {
    'name': 'abc',
    'pass': '123456'
}

# 把提交的数据用urlencode编码下, 防止里面的中文乱码
post_data = urllib.parse.urlencode(post_origin_data).encode('utf-8')

# 构建请求头
xxxx_req = urllib.request.Request(url, headers=head, data=post_data)

# 发送请求并接收响应
xxxx_resp = urllib.request.urlopen(xxxx_req).read()

# 解码响应
xxxx_data = xxxx_resp.decode('utf-8')

print("post result: %s" % xxxx_data)
```

可用以下等价命令来查看细节

> curl -v -d "name=abc" -d "pass=1234567" http://www.iqianyue.com/mypost

可以看出POST与GET方法的区别就是

- 如果把数据都放在url里提交就是GET
- 把数据用data参数提交就是POST

总结下整个过程:

- 导入相关库
- 构建Requst对象(接口地址, 请求头设置, 请求体创建/编码)
- 发送请求 (urlopen)
- 对响应进行解码 (decode)

### 设置Header

有时我们有对请求头添加些特殊配置的需求 

比如有个接口是 `http://e.flyme.cn/ad/get`, 这个接口有测试环境, 有异地环境, 有预发布环境, 有正式环境

直接请求, 一般是到正式环境; 如果我们想指定请求环境, 一般是要在客户端设置host

如果脚本换台机器执行, 还要重新配置/etc/hosts, 这虽然很麻烦但至少能解决问题;

换一个场景, 如果线上有16台服务器, 我们需要对每台机器这个接口进行监控,

问题的关键是怎么让请求到指定的机器上, 即外网的16台服务器每次都一个不落的请求一次

总不能请求完一台服务器, 就去修改下/etc/hosts吧

解决的方法就是:

- 请求头加上`host`
- 接口地址用IP

样例代码如下:

```python
import urllib.request

url = 'http://183.232.2.130/appstore/search.do?keyword=今日头条&imei=865964033244675&device_model=M5Note&net=4g'
# 设置Header里的host, 如果有其它需要; 按规范加到head里即可
head = {'host': 'bro.flyme.cn'}

hot_req = urllib.request.Request(url, headers=head)

hot_resp = urllib.request.urlopen(hot_req).read()

hot_data = hot_resp.decode('utf-8')

print("data: %s" % hot_data)

```

可用以下等价命令来查看请求细节

> curl -v -H "host: bro.flyme.cn" http://183.232.2.130/appstore/search.do?keyword=今日头条&imei=865964033244675&device_model=M5Note&net=4g

### 设置超时时间

urllib的Http请求默认没的timeout时间, 一般由服务端断开

当是有时我们需要从客户端主动断开连接, 比如超过3秒没有响应就不再等待了

```python
import socket

# 里面的参数即可设置的超时等待秒数
socket.setdefaulttimeout(3)
```
