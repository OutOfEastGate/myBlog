---
title: java学习日记
date: 2022-03-20 18:37:02
tags:
---

java学习路线：

java基础：韩顺平

尚硅谷：spring5=》springMVC=》springboot

git

linux

# java基本知识
- Class（类）是java中最小单元，java所有内容都是放在类中的
- 每一个.java文件都是由类组成
- 一个java文件可以有多个class(不提倡),但只能有一个public class
- public class的名字必须和文件名字一样，大小写都要完全一致
- main函数的基本格式PSVM

        public static void main(String[] args)

- 自定义函数调用方法

        public class hello {
            public static void main(String[] args){
            System.out.println("hello");
            hello.in();
        }
            public static void in(){//自定义函数在调用时才会执行
            System.out.println("in函数");
        }
        }
# java 进阶
****
## junit
单元测试

    @Test
    public void testAdd(){
        int a=1;
        int b=2;
        int c=a+b;
        assert.assertEquals(3,c);//断言
    }

在所有方法执行前都会自动执行

@Before

在所有方法结束后都会执行

@After

## 反射
反射：将类的各个部分封装为其他对象，这就是反射机制 

.java文件被编译成字节码文件.class是类加载器会把成员变量封装进Filed ，构造方法会被封装进Constructor中，成员方法也会被封装进Method

- 好处
  - 可以在程序运行过程中操作这些对象
  - 可以解耦，提高程序的可扩展性

获取class对象的方式：
- class.forName("全类名")：将字节码文件加载进内存，返回class对象
  - 多用于配置文件
- 类名.class:通过类名的属性class获取
  - 多用于参数传递
- 对象.getClass()  getClass()方法在Object类中定义着
  - 多用于对象的获取字节码的方式 

