---
title: JavaWeb
date: 2022-04-07 17:45:16
tags: java
catecories: 学习笔记--javase/javaee
---
<h1> JavaWeb</h1>

## TomCat服务器
****
Tomcat 服务器是一个免费的开放源代码的Web应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP程序的首选

## Http
****
(超文本传输协议）是一个简单的请求-响应协议，它通常运行在TCP之上。

### HTTP 的请求
- get：请求能够携带的参数比较少，大小有限制，会在浏览器的URL地址栏显示数据内容，不安全，但高效

- post:请求能够携带的参数没有限制，大小没有限制，不会在浏览器的URL地址栏显示数据内容，安全，但不高效

 HTTP 的请求报文结构：
![](https://img-blog.csdnimg.cn/20190407164610965.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MTgyMTI1,size_16,color_FFFFFF,t_70)

- 请求行：由请求方法（Method）、URL 字段和 HTTP 的协议版本组成，注意其中的空格、回车符和换行符均不可省略，所以我们的请求方法实际上就是位于请求行中的了。
- 请求头部：位于请求行之后，个数可以为 0~若干个，每个请求头部都包含一个头部字段名和一个值，它们之间用冒号 ":" 分隔，在最后用回车符和换行符表示结束。
- 请求数据：如果请求方法为 GET，那么请求数据为空。它主要是在 POST 中进行使用，适用于需要填表单（FORM）的场景。


请求方法：Get,Post,HEAD,DELETE,PUT,TRACT.…

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTUxNTExMS8yMDIwMDEvMTUxNTExMS0yMDIwMDExMDIwMTQzMDE5My0xNTU5MTYwNTg4LnBuZw?x-oss-process=image/format,png)





