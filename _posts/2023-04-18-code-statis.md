---
layout: post
title: "统计git代码提交量"
subtitle: "查看有没有摸鱼"
date: "2023-04-18 23:59"
author: "mingo"
header-img: "assets/img/post-bg-web.jpg"
tags: []
---

## 功能说明

统计当前项目代码分别由哪些人提交，新增/删除/累计行数

其中`累计行数=新增-删除`，有可能是负数

## 实现

```bash
#! /bin/bash

printf "%10s\t%10s\t%10s\t%10s\n" "name" "add" "remove" "total";
printf "%s--------------------------------------------------------------\n" "";


git log --format='%aN' |sort -u  | while read name; 
do
    git log --author="$name" --pretty=tformat: --numstat |\
    # 过滤掉自动生成的代码,第三方引入的代码
    grep -v "stub/" |grep -v "vendor/" | \
    awk  '{ 
        add += $1; 
        subs += $2; 
        loc += $1 - $2 
    } END { 
        printf "%10s\t%10s\t%10s\t%10s\n", username, add, subs, loc 
    }' username="${name}" -;
done
```

### 使用方法

把上述脚本保存在`code_statis.sh`文件中，然后在git工程目录下执行即可

```shell
bash ${bash_path}/code_statis.sh
```

输出效果：
```shell
➜  myproj git:(master) bash ~/code_statis.sh
      name             add          remove           total
--------------------------------------------------------------
 axcdzhang             599              88             511
  dxefjliu            3701            1042            2659
  dddddxin            1032              26            1006
```