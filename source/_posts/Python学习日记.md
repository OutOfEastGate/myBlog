---
title: Python从入门到入睡
date: 2022-06-05 16:24:34
tags: Python
categories: 学习笔记-工具
cover: https://images.pexels.com/photos/1072179/pexels-photo-1072179.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2
---

## Python基本数据类型

### 字符串

{%folding 字符串%}

声明：

```python
x = "Hello world"
```

打印：

```python
print(x)
```

打印第一个字符和最后一个字符

```python
print(x[0])
print(x[-1])
```

打印从0到5个字母

```python
print(x[0:5])
```

转义字符：输出结果为Hello" world

```python
x = "Hello\" world"
```

格式化字符串：

```python
first_name = "wang"
last_name="hongtao"
full=f"{first_name} {last_name}"#或者full=first_name+last_name
```

{%note simple info%} f代表format，大括号中可以是字符也可以是其他数据类型 {%endnote%}

字符串相关方法：

```python
print(full.upper())#将字母全部大写
print(full.lower())#将字母全部小写
print(full.title())#将所有首字母大写
print(full.rstrip())#right strip去除结尾换行、回车、制表符、空格
print(full.lstrip())#right strip去除开头换行、回车、制表符、空格
print(full.find("h"))#找到字符索引
print(full.replace("h","H"))#替换字符
print("wang" in full)#查找字符串中是否有指定字符串
```

{%endfolding%}

### 数字

{%folding 字符串%}

算数基本操作

```python
print(10+3)
print(10-3)
print(10*3)
print(10/3)
print(10//3)#整数除法
print(10 % 3)
print(10**3)#求次方
```

python 中math的库函数： https://docs.python.org/zh-cn/3/library/math.html

类型转换：

```python
x=input("x=")
y=int(x)+1
print(f"x:{x},y:{y}")
```

常用转换：

```python
int(x)
float(x)
bool(x)
str(x)
```



{%endfolding%}

## 控制流

{%folding green,if 字句%}

if else 格式：

```sql
x=6
if x==5:
    message="x=5"
elif x==6:
    message="x=6"
else:
    message="x!=5"
print(message)
```

简便形式：三元运算符

```sql
message="x=5" if x==5 else "x!=5"
```

{%note simple info%}

在有多个条件判断时，连接符有and、or和and not

也可以使用下面这种形式进行判断

{%endnote%}

```sql
x=6
if 1<=x<=7:
    print("Yes")
```

{%endfolding%}



{%folding green,循环字句%}

```python
for num in range(10):
    print("number:",num)
```

输出结果：

{%folding ,输出结果%} 

number: 0
number: 1
number: 2
number: 3
number: 4
number: 5
number: 6
number: 7
number: 8
number: 9

{%endfolding%}

```python
for num in range(1,10):
    print("number:",num)
```

{%folding ,输出结果%} 

number: 1
number: 2
number: 3
number: 4
number: 5
number: 6
number: 7
number: 8
number: 9

{%endfolding%}

```python
for num in range(1,10,2):
    print("number:",num)
```

{%folding ,输出结果%} 

number: 1
number: 3
number: 5
number: 7
number: 9

{%endfolding%}

{%note simple info%}

跳出循环可以使用break字句

{%endnote%}

在Python中执行print(type(range(5)))的结果是“<class 'range'>”

所以in后面可以放任意值，比如：

```python
for num in "range":
    print("number:",num)
```

{%folding ,输出结果%} 

number: r
number: a
number: n
number: g
number: e

{%endfolding%}

{%note simple info%}

in后面同样可以放列表，也可以放自定义的对象

{%endnote%}

以上是for循环，下面是while循环

```python
command = ""
while command !="quit":
    command = input(">")
    print("ECHO: ",command)
```

{%folding ,输出结果%} 

>1
>ECHO:  1
>2
>ECHO:  2
>quit
>ECHO:  quit
>PS D:\PythonProject> 

{%endfolding%}

{%endfolding%}

{%folding ,自定义函数%} 

使用def关键字定义函数

```python
def greet(first_name,last_name):#定义
    print(f"{first_name},{last_name}")
greet("wang","hongtao")#调用
```

带有返回值的函数

```python
def increment(number,by):
    return number + by
print(increment(2,3))
```

可变参数列表函数

```python
def multiply(*numbers):
    for number in numbers:
        print(number)
multiply(2,3,4,5)
```

{%folding ,输出结果%} 

2
3
4
5

{%endfolding%}

```python
def fun(**user):
    print(user["id"])
    print(user["name"])
fun(id=1,name="wht")
```

{%folding ,输出结果%} 

1
wht

{%endfolding%}

{%endfolding%}

## 数据结构

{%folding ,list%} 

```python
list1 = [[0, 1], [2, 3], 'wang']
list2 = ['hong', 'tao']
list3 = list1+list2
numbers = list(range(10))
chars = list("wang")
print(list3)
print(numbers)
print(chars)
```

链表中的元素可以是任意的，既可以是数组也可以是字符串，也可以是list，list和list之间可以相加

{%folding ,输出结果%} 

[[0, 1], [2, 3], 'wang', 'hong', 'tao']
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
['w', 'a', 'n', 'g']

{%endfolding%}

**对链表元素的操作：**

```python
list1 = [[0, 1], [2, 3], 'wang']
list2 = ['hong', 'tao']
list3 = list1+list2

print(list3[0])
print(list3[-1])
```

输入-1代表列表的最后一个元素





{%folding ,输出结果%} 

[0, 1]
tao

{%endfolding%}

**对数字链表的操作：**

```python
list1=list(range(20))
print(list1[0:3])
print(list1[::2])
print(list1[::-1])
```

可以使用`list1[::?]`指定步数，当？=-1时，代表链表反方向输出



{%folding ,输出结果%} 

[0, 1, 2]
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
[19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0]

{%endfolding%}

```python
list1=list(range(3))
first,second,third=list1
print(f"{first},{second},{third}")
```

链表的值赋给单个变量，必须让变量的数量等于链表的元素个数，否则会报错

可以在最后指定*other代表其他元素赋给other这个元素



输出结果：`0,1,2`

```python
list1=list(range(10))
first,second,*other=list1
print(f"{first},{second},{other}")
```

输出结果：`0,1,[2, 3, 4, 5, 6, 7, 8, 9]`

{%note simple info%}

在python中，带有`*`号的变量一般指可以是任意变量，带有两个`*`的变量指可以任意个数的任意变量

{%endnote%}

**对链表进行遍历：**

普通遍历方式：

```python
items=[0,'a','wang']
for item in items:
    print(item)
```

输出结果:`0 a wang`

enumerate将结果包装并带上索引：

```python
items=[0,'a','wang']
for item in enumerate(items):
    print(item)
```

输出结果：

```
(0, 0)
(1, 'a')
(2, 'wang')
```

以上方式可以换一种输出方式：

```python
items=[0,'a','wang']
for index,item in enumerate(items):
    print(index,item)
```

输出结果：

```
0 0
1 a
2 wang
```

**对链表增删改**：

```python
items=[0,'a','b','c']
items.append("d")#在list的结尾追加一个d
items.insert(1,'1')#在链表的index为1的地方插入1
items.remove("b")#删除list中的b
del items[0:3]#删除list中index从0到3的元素
items.clear()#清空list
```

**对list进行排序：**

```python
numbers=[4,6,3,1,9,43]
numbers.sort(reverse=False)#会改变原有的list
numbers2=sorted(numbers,reverse=True)#不会改变原有的list
```

当list中的元素并不是纯数字时：

```python
items=[
    ("item1",10),
    ("item2",9),
    ("item3",12)
]
items.sort(key= lambda item:item[1])
print(items)
```

lambda表达式在过滤中的应用：

```python
items=[
    ("item1",10),
    ("item2",9),
    ("item3",12)
]
filtered = list(filter(lambda item:item[1]>=10,items))
print(filtered)
```

上面的例子筛选大于等于10的元素，但是有一种更简便的方法，那就是下面的例子





```python
items=[
    ("item1",10),
    ("item2",9),
    ("item3",12)
]
filtered = [item for item in items if item[1]>=10] #基本语法[expression for item in items if expression]
print(filtered)
```



{%endfolding%}

