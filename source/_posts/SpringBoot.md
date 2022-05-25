---
title: SpringBoot
date: 2022-04-16 20:55:33
tags: 学习笔记
category: spring
---
# SpringBoot
## Spring 缺点
1） 配置繁琐
虽然Spring的组件代码是轻量级的，但它的配置却是重量级的。一开始，Spring用XML配置，而且是很多
XML配置。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。
Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。
所有这些配置都代表了开发时的损耗。因为在思考Spring特性配置和解决业务问题之间需要进行思维切换，所
以编写配置挤占了编写应用程序逻辑的时间。和所有框架一样，Spring实用，但它要求的回报也不少。

2）依赖繁琐
项目的依赖管理也是一件耗时耗力的事情。在环境搭建时，需要分析要导入哪些库的坐标，而且还需要分析导
入与之有依赖关系的其他库的坐标，一旦选错了依赖的版本，随之而来的不兼容问题就会严重阻碍项目的开发
进度。

----
## SpringBoot 概述
### SpringBoot 功能
1） 自动配置
Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定
Spring配置应该用哪个，不该用哪个。该过程是SpringBoot自动完成的。

2） 起步依赖
起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖
，这些东西加在一起即支持某项功能。
简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

3） 辅助功能
提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等。
Spring Boot 并不是对 Spring 功能上的增强，而是提供了一种快速使用 Spring 的方式。

----
## SpringBoot 起步依赖原理分析

- 在spring-boot-starter-parent中定义了各种技术的版本信息，组合了一套最优搭配的技术版本。
- 在各种starter中，定义了完成该功能需要的坐标合集，其中大部分版本信息来自于父工程。
- 我们的工程继承parent，引入starter后，通过依赖传递，就可以简单方便获得需要的jar包，并且不会存在
版本冲突等问题。

----
## SpringBoot 配置
### 配置文件分类
SpringBoot是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用
application.properties或者application.yml（application.yaml）进行配置。
-  properties：

    server.port=8080
-  yml: 
   server:

    port: 8080
### YAML
YAML全称是 YAML Ain't Markup Language 。YAML是一种直观的能够被电脑识别的的数据数据序列化格式，并且容易被人类阅
读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP
等。YML文件是以数据为核心的，比传统的xml方式更加简洁。
YAML文件的扩展名可以使用.yml或者.yaml。

### YAML：基本语法
-  大小写敏感
-  数据值前边必须有空格，作为分隔符
-  使用缩进表示层级关系
-  缩进时不允许使用Tab键，只允许使用空格（各个系统 Tab对应的 空格数目可能不同，导致层次混乱）。
-  缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
-  '#' 表示注释，从这个字符一直到行尾，都会被解析器忽略。
```yaml
server: 
port: 8080
address: 127.0.0.1
name: abc
```

### YAML：数据格式
-  对象(map)：键值对的集合。
```yaml
person:
name: zhangsan
# 行内写法
person: {name: zhangsan}
```
- 数组：一组按次序排列的值
```yaml
address:
- beijing
- shanghai
# 行内写法
address: [beijing,shanghai]
```
- 纯量：单个的、不可再分的值
```yaml
msg1: 'hello \n world' # 单引忽略转义字符
msg2: "hello \n world" # 双引识别转义字符
```
### YAML：参数引用
```yaml
name: lisi
person:
name: ${name} # 引用上边定义的name值
```
----
>小结
>
>1） 配置文件类型
>
> properties：和以前一样
> 
> yml/yaml：注意空格
> 
>2） yaml：简洁，以数据为核心
>
> 基本语法
> 
>• 大小写敏感
>
>• 数据值前边必须有空格，作为分隔符
>
>• 使用空格缩进表示层级关系，相同缩进表示同一级>
>
>数据格式
>
>• 对象
>
>• 数组: 使用 “- ”表示数组每个元素
>
>• 纯量
>
> 参数引用
> 
>• ${key}
## SpringBoot注入yml
### 使用value注解
```java
    @Value("${name}")//元素注入
    private String name1;
    @Value("${person.age}")//对象注入
    private int age;
    @Value("${address[0]}")//数组注入
    private String address;
```
### 使用Environment获取yml对象
```java
//使用Environment获取yml对象
    @Autowired
    Environment environment;
    //在方法中
    String name = environment.getProperty("address");
```
### ConfigurationProperties
```java
@Component
@ConfigurationProperties("person")
public class Person {
    private String name;
    private int age;
```

## profile
我们在开发Spring Boot应用时，通常同一套程序会被安装到不同环境，比如：开发、测试、生产等。其中数据库地址、服务
器端口等等配置都不同，如果每次打包时，都要修改配置文件，那么非常麻烦。profile功能就是来进行动态配置切换的。

1） profile配置方式

-  多profile文件方式

-  yml多文档方式

2） profile激活方式

-  配置文件

- 虚拟机参数

-  命令行参数

