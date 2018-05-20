---
title        : Python学习(3) - 工具类
author       : ahcming
description  : 
date         : 2018-05-20 16:59
layout       : post
tag          : ["work", "note", "python"]
blog         : true
---

## Json

通过json库我们可以把一个对象转为格式是json的字符串

也可以把格式是json的字符串转为对象

```python
import json

person = {'name':'cm', 'age':18, 'like': ['programing', 'read', 'daze']}

# type(person)  -> <class 'dict'>

# dumps把一个对象变成json 
ss = json.dumps(person)
# '{"name": "cm", "age": 18, "like": ["programing", "read", "daze"]}'
# type(ss)      -> <class 'str'>

# loads把一个字符串变成对象
js = json.loads(ss)
# type(js)     -> <class 'dict'>

js['name']
# 'cm'
```

## 时间格式化

在计算机内部时间是一个比较大的数字, 代表当前时间与`1970-01-01 00:00:00`的时间差(单位秒)

比如: 1526807661.936172

但是这不利于人直接观察理解, 所以一般需要转成`2018-05-20 14:15:16`这样的格式 

```python
import time

# 获取当前时间的精确值
time.time()

# 获取当前时间的秒数
int(time.time())

# 当前时间的tuple格式
# time.struct_time(tm_year=2018, tm_mon=5, tm_mday=20, tm_hour=17, tm_min=22, tm_sec=11, tm_wday=6, tm_yday=140, tm_isdst=0)
time.localtime()

# 把当前时间转成指定格式, 比如YYYY-MM-dd HH:mm:ss
time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())

# 把一个字符串时间转成时间对象(time tuple格式)
# time.struct_time(tm_year=2018, tm_mon=5, tm_mday=20, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=6, tm_yday=140, tm_isdst=-1)
time.strptime('2018-05-20', '%Y-%m-%d')
```

## 文件读写


```python
conf = open('/data/conf/grape/grape-engine/V1.0')

# 一次性全部读取
print(conf.read())

# 或按行读取
for line in conf.readlines():
    print(line)

conf.close()
```

文件读取3步:

- open: 指定文件路径
- read/readline/readlines: 读取文件内容
- close: 必须close, 不然会有文件句柄泄漏的严重错误

关于read常见有3种方法:

- read(): 一次把整个文件读进内存; 当文件比较小时可用此方法
- readline(): 按行读取, 一次读一行, 性能较差
- readlines(): 读取整个文件并返回一个list(每行一个元素), 最常用做法


```python
# 指定要输出的文件目录地址和写的模式
# w: 覆盖写, 每次会把文件里的内容覆盖掉
# a: 追加写, 每次在文件最后添加新内容
cp_conf = open('/tmp/engine_v1.0', 'w')

cp_conf.write('this is a important config\n')

# 把内存里的数据刷新到硬盘
cp_conf.flush()

# 如果没有调用flush, 会在close里把内存里的数据刷新到硬盘
cp_conf.close()
```

文件写入3步;

- open: 指定要写的文件路径
- write: 要写的内容, 要自己指定换行符
- flush: 把数据刷新到硬盘, 非必须
- close: 关闭文件句柄

