---
title: Spring源码
date: 2022-09-05 20:47:45
tags: spring
category: 学习笔记-框架
---

## 第一讲 `BeanFactory`与`ApplicationContext`的区别与联系

**到底什么是`BeanFactory`？**

```java
ConfigurableApplicationContext context = SpringApplication.run(SpringBootTestApplication.class, args);
```

它是`ApplicationContext`的父接口

![](https://s1.ax1x.com/2022/09/05/v7AzvV.png)

>  鼠标选中`ConfigurableApplicationContext`，按`Ctrl + Shift + U`或者`Ctrl + Alt + U`打开类图，可以看到`ApplicationContext`的有个父接口是`BeanFactory`

`BeanFactory`才是 `Spring` 的核心容器，`ApplicationContext`的主要实现都 [组合]了他的功能

执行下面的代码，打印context

```java
ConfigurableApplicationContext context = SpringApplication.run(SpringBootTestApplication.class, args);
System.out.println(context.getClass());
```

执行结果：

```java
class org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext
```

可以看到SpringBoot的启动程序返回的`ConfigurableApplicationContext`的具体的实现类是`AnnotationConfigServletWebServerApplicationContext`

进入`AnnotationConfigServletWebServerApplicationContext`的类图中，发现其继承自`GenericApplicationContext`

![](https://s1.ax1x.com/2022/09/05/v7ETR1.png)

按F4进入`GenericApplicationContext`，

```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
    private final DefaultListableBeanFactory beanFactory;
    @Nullable
    private ResourceLoader resourceLoader;
    private boolean customClassLoader;
    private final AtomicBoolean refreshed;
```

在这个类里面可以找到`beanFactory`作为成员变量出现

`BeanFactory`中的方法

<img src="https://s1.ax1x.com/2022/09/05/v7ENKf.png" width="60%;" />

```
org.springframework.beans.factory.support.DefaultListableBeanFactory@53f6fd09
```

