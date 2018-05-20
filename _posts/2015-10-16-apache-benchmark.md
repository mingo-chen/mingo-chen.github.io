---
layout: post
title: 性能测试工具－ab
category : bottleneck
tag : ["work", "original", "ab"]
blog: true
---
ab的全称是ApacheBench,是Apache附带的一个小工具
专门用于HTTP Server的benchmark testing,可以同时模拟多个并发请求

{{ site.excerpt_separator }}

## 经典用法
* 指定并发数与请求总数

> ab -n 10000 -c 100 "http://xxxx.xxx.com/a/b/c.do"

> -n用于指定总请求数, -c用于指定并发数

这句话的作用是模拟100个用户对接口http://xxxx.xxx.com/a/b/c.do 发起10000个请求

* 指定并发数与请求时间

> ab -c 100 -t 10 "http://xxxx.xxx.com/a/b/c.do"

> -t 表示总共请求10秒

这句话的作用时模拟100个用户对接口http://xxxx.xxx.com/a/b/c.do 持续10秒的请求

* 指定请求头

> ab -H "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-Us) AppleWeb Kit/534.2 (KHTML, like Gecko) Chrome/6.0.447.0 Safari/534.2"

执行结果

```
ab -n 100000 -c 5000 -r http://www.reader.duowan.com/unauth/settings/get.do   
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.reader.duowan.com (be patient)
Completed 10000 requests
Completed 20000 requests
Completed 30000 requests
Completed 40000 requests
Completed 50000 requests
Completed 60000 requests
Completed 70000 requests
Completed 80000 requests
Completed 90000 requests
Completed 100000 requests
Finished 100000 requests


Server Software:        nginx
Server Hostname:        www.reader.duowan.com
Server Port:            80

Document Path:          /unauth/settings/get.do
Document Length:        70 bytes

Concurrency Level:      5000
Time taken for tests:   22.484 seconds
Complete requests:      100000
Failed requests:        828
   (Connect: 0, Receive: 276, Length: 276, Exceptions: 276)
Write errors:           0
Total transferred:      20705060 bytes
HTML transferred:       7035700 bytes
Requests per second:    4447.52 [#/sec] (mean)
Time per request:       1124.222 [ms] (mean)
Time per request:       0.225 [ms] (mean, across all concurrent requests)
Transfer rate:          899.28 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       19  368 692.8    186    9439
Processing:    10  681 1012.3    340   13431
Waiting:        0  576 967.5    244   13337
Total:         71 1049 1229.8    635   13587

Percentage of the requests served within a certain time (ms)
  50%    635
  66%    779
  75%    985
  80%   1306
  90%   3292
  95%   3708
  98%   4385
  99%   5242
 100%  13587 (longest request)
```

结果说明:

- 其中最关键的结果就是`Requests per second:    4447.52 [#/sec] (mean)`,传说中的TPS
- 接口耗时,`Time per request`第一个是并发组所耗总时,第二个是单个接口耗时

## 一些问题
当单机并发数过大时，ab会报错误(apr_socket_recv:connection reset by peer), 该问题原因是由于测试机文件句柄数不够多导致<br/>
并发数跟*Unix的内核文件句柄数相关,修改服务器配置以增大并发数<br/>
查询当前服务器的文件句柄数:<br/>
> ulimit -a 
> 或
> ulimit -n

执行结果:

```
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
file size               (blocks, -f) unlimited
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 256
pipe size            (512 bytes, -p) 1
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 709
virtual memory          (kbytes, -v) unlimited
```

其中open files就是最大文件句柄数<br/>
增加文件句柄数
> ulimit -n 10240

或 vi /etc/secrity/limits.conf<br/>

增加2行

```
*	soft	nofile	10240
*	hard	nofile	10240
```

## 参考资料
- [api说明](http://httpd.apache.org/docs/2.0/programs/ab.html)
- [ulimit说明](http://blog.yufeng.info/archives/1380)
- [APACHE BENCHMARK](http://mo2g.com/view/38/)
