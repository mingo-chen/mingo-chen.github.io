---
layout: post
title: freemarker模板语言笔记

category : java, freemarker,
tag : ["work", "note", "freemarker"]
blog: true
---
# freemarker模板语言
{{ site.excerpt_separator }}

## 定义变量 ##

> <#assign text="hello, ${name}" 

## 分支选择 ##

```html
<#if (age>40)>
older...
<#elseif (age>18)>
younger..
<#else>
smaller...
</#if>
```

## 算术,逻辑运算 ##

> 同JAVA一样

## 从文件创建实例 ##

```html
String ftlPath = FreemarkerEngine.class.getResource("/ftl").getPath();
Configuration configuration = new Configuration();

configuration.setTagSyntax(Configuration.ANGLE_BRACKET_TAG_SYNTAX);
configuration.setDirectoryForTemplateLoading(new File(ftlPath));

Template template = configuration.getTemplate("hello.ftl");
Map<String, Object> datas = new HashMap<String, Object>();
datas.put("name", "cm");
datas.put("age", 19);

Writer out = new OutputStreamWriter(System.out);
template.process(datas, out);
out.flush();
```

## 从string创建实例 ##

```html
Configuration configuration = new Configuration();
configuration.setTagSyntax(Configuration.SQUARE_BRACKET_TAG_SYNTAX);

configuration.setSharedVariable("cp", "mz");

String content = "name:${name}\r\nage:${age}\r\n==>${cp}, ${caller(\"cm\", name, age)}";
Template template = new Template("first", content, configuration);

Map<String, Object> datas = new HashMap<String, Object>();
datas.put("name", "cm");
datas.put("age", 18);

Writer out = new OutputStreamWriter(System.out);
template.process(datas, out);
out.flush();
```

## 自定义方法 ##
先实现方法

```html
public class FreemarkerCaller implements TemplateMethodModelEx {

    @Override
    public Object exec(List arguments) throws TemplateModelException {
        System.out.println("==>开始执行自定义方法");

        for(Object value : arguments) {
            System.out.println("-->" + value);
        }
        return "succ";
    }
}

```

然后通过参数对象传递到模板中

```html
datas.put("caller", new FreemarkerCaller());
```

在模板中使用

> ${caller(\"cm\", name, age)} // 如果调用其它变量, 直接写变量名即可, 比如此处的name, age都是变量
