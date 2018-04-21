---
layout: post
tagline: "重思想,轻代码"
title: thrift学习笔记

category : thrift
tags : [thrift, 学习笔记]
---

## thrift基本入门

### 接口定义

    service HelloThrift {
		  void sayHello(1:string message);
		
		  i16 say(1:string message, 2:i16 length);
    }

{{ site.excerpt_separator }}

每个接口可分为3部分
- 接口名(sayHello, say)
- 接口参数(1:string message, 2:i16 length)
- 接口返回值(void, i16)

### Client与Server通信交互过程
具体对于say接口来说,会生成一个say对象
Client向Server发送消息就是send_say(parmeters...);
Client接收Server消息就是recv_say();
{% gist 9635417 %}	

首先先解释下,以下对象的作用
- **TBase**: 对应一个接口
- **TMessage**: 
- **TStruct**:
- **TFiled**: 封装接口的参数,返回值的类型信息
    id => 参数在参数列表中的序号
    type => 参数类型
    name => 参数名

### 数据传输
实现原码如下:
{% gist 9635434 %}

可见其数据格式为:

	message begin
		struct begin
			filed begin
				filed content
			filed end
			...
      ...
			filed stop
		struct end
	message end

- **TProtocol 协议层**
    决定C/S之间信息传递的格式,有:
    TBinaryProtocol(二进制)
    TCompactProtocol(紧凑格式,压缩的)
    TTupleProtocol()
    TJSONProtocol(Json格式)
    TSimpleJSONProtocol()

- **TTransport 传输层**
    决定C/S之间信息传递的方式,有:
    TFramedTransport
    TNonblockingSocket(非阻塞方式)
  
