---
title: css学习日记
date: 2022-03-21 19:33:08
tags: 前端
category: 前端
---
# css简介
css是层叠样式表（cascading style sheets）

css是用来美化网页页面的

# css语法规范
css由两大块组成，分别是<strong>选择器和样式</strong>

style的位置应该在head里

    <head>
    <style>
        p {
            <!-- 修改文字为红色 -->
            color: red
            <!-- 修改字体大小 -->
            font-size: 12px
        }
    </style>
    </head>
    <body>
        <p>颜色</p>
    </body>
## css代码风格
- 样式格式
展开式

        p {
        <!-- 修改文字为红色 -->
            color: red
        <!-- 修改字体大小 -->
            font-size: 12px
        }

- 样式大小写：
一般用小写
- 空格规范
属性值前面和冒号后面，保留一个空格，选择器和大括号中间保留空格

 # css基础选择器
 ## css选择器分类
 基础选择器、复合选择器

 基础选择器分为：标签选择器、类选择器、id选择器、通配符选择器

### 标签选择器
以html的标签名作为选择器，按照标签名分类，为页面中同一类别的标签统一css格式

例如：将p标签改为绿色，div标签改为红色

    <style>
        p {
            color: green;
        }
        div {
            color: red;
        }
    </style>

### 类选择器
可以差异化选择不同的标签，单独选一个或者某几个标签

例如，将所有拥有red类的html元素均为红色

    .red {
        color: red;
    }
在html中给某个标签添加class属性即可

    <li class="red"> 红色</li>
#### 类选择器-多类名
标签可以选择多个类名，中间必须用空格隔开

    <div class="lei1 lei2"> </div>
### id选择器
定义格式

    #idname {
        color: red;
    }
调用格式

    <li id="idname"></li>
id选择器与类选择器的区别：id选择器定义方式不同，且<strong><em>只能调用一次</strong></em>

### 通配符选择器
在css中，通配符选择器表示选取页面中所有标签

    *{
        属性值1：属性值1；
        ...
    }
通配符选择器不需要调用，会自动调用
# css复合选择器
## 后代选择器
后代选择器用于选取某元素的后代元素。

    div p
    {
    background-color:yellow;
    }
## 子元素选择器
与后代选择器相比，子元素选择器（Child selectors）只能选择作为某元素直接/一级子元素的元素

    div>p
    {
    background-color:yellow;
    }
## 相邻兄弟选择器
选取了所有位于 div 元素后的第一个 p 元素

    div+p
    {
    background-color:yellow;
    }

## 后续兄弟选择器
后续兄弟选择器选取所有指定元素之后的相邻兄弟元素


    div~p
    {
    background-color:yellow;
    }

# css字体属性
## 字体系列
    <style>
    P {
        font-family: '宋体',Times,serif
    }
    </style>

当有多个字体时，系统会默认使用第一种，如果第一种不可用在用后面的字体

字体大小

    p {
        font-size: 20px
    }
字体粗细

    p {
        font-weight: bold 或 700
    }

### 文字样式

文字斜体

    P {
        font-style: italic
    }
文字不倾斜

    p {
        font-style: normal
    }

文字复合属性

font-style font-weight font-size font-family 顺序不可颠倒

    p {
        font: italic 700 16px 'Microsoft yahei'
    }
# css文本属性
## 文本颜色颜色

    <style>
        div {
            <!-- color: pink  -->
            <!-- color: #ff0000 -->
            <!-- color: rgb(255,0,0) -->
        }
    </style>
## 对齐文本

    div {
        text-align: center; right left
    }
## 装饰文本

    div {
        text-decoration: underline;//下划线
        text-decoration: line-through;//删除线
        text-decoration: none;//默认没有装饰线（用的最多）
    }
## 文本缩进

    p {
        text-indent: 20px //文本首行缩进
        text-indent: 2em //常用单位，1em就是缩进一个文字大小
    }
## 行间距

    p {
        text-height: 26px;//是包括文字和上下间距
    }
# css引入方式
## 内部样式表
css写在html内部，放在style标签内部

一般用于控制整个html页面
## 行内样式表
直接在标签内部修改样式

    <p style="color:red">  红色</p>
<p style="color:red ;font-size:20px">  红色</p>
一般不推荐使用

## 外部样式表
单独写一个css文件，引入到html中

引入方式:

    <link rel="stylesheet" href="01.css">//快捷键:link+tab
# css盒子

<img src="https://www.runoob.com/images/box-model.gif">

    div {
        width: 300px;
        border: 25px solid green;
        padding: 25px;
        margin: 25px;
    }
border-style :
none: 默认无边框

dotted: 定义一个点线边框

dashed: 定义一个虚线边框

solid: 定义实线边框

double: 定义两个边框。 两个边框的宽度和 border-width 的值相同

groove: 定义3D沟槽边框。效果取决于边框的颜色值

ridge: 定义3D脊边框。效果取决于边框的颜色值

inset:定义一个3D的嵌入边框。效果取决于边框的颜色值

outset: 定义一个3D突出边框。 效果取决于边框的颜色值
<style>
    p.none{
        border-style:none;
    }
    p.dotted{
        border-style:dotted;
    }
    p.dashed{
        border-style:dashed;
    }
    p.ridge{
        border-style: ridge;
    }
    p.ridge2{
        border-style: ridge;
        border-color:red;
        border-width:5px

    }
    p.ridge3{
        border-style: ridge;
        border-color:red;
        border-width:5px;
        outline-style: ridge;
        outline-color:green
        
    }



</style>
## css边框-border
<div>
<p class="none"> none</p>
<p class="dotted"> dotted</p>
<p class="dashed"> dashed</p>
<p class="ridge"> ridge</p>
</div>
指定边框宽度和边框颜色

        p.ridge2{
            border-style: ridge;
            border-color:red;//必须在style后面
            border-width:5px
        }
<p class="ridge2"> ridge2</p>

## css轮廓-outline

指定轮廓样式和颜色

    p.ridge3{
        border-style: ridge;
        border-color:red;
        border-width:5px;
        outline-style: ridge;
        outline-color:green
        
    }
<p class="ridge3"> ridge3</p>

## 外边距-margin 填充-padding
<img src="https://www.runoob.com/wp-content/uploads/2013/08/VlwVi.png">

# css可视

指定h1不可见，但依然占有空间

    h1.hidden {
        visibility:hidden;
        }
# css定位

    static
    relative
    fixed
    absolute
    sticky
# css伪类

    a:link {color:#FF0000;} /* 未访问的链接 */
    a:visited {color:#00FF00;} /* 已访问的链接 */
    a:hover {color:#FF00FF;} /* 鼠标划过链接 */
    a:active {color:#0000FF;} /* 已选中的链接 */

# css伪元素


    p:first-letter 
    {
        color:#ff0000;
        font-size:xx-large;
    }
first-letter表示首字母 first-line表示首行
## css before元素

    h1:before 
    {
        content:url(smiley.gif);
    }
# css导航栏


    <ul>
    <li><a href="#home">主页</a></li>
    <li><a href="#news">新闻</a></li>
    <li><a href="#contact">联系</a></li>
    <li><a href="#about">关于</a></li>
    </ul>
<ul>
  <li><a href="#home">主页</a></li>
  <li><a href="#news">新闻</a></li>
  <li><a href="#contact">联系</a></li>
  <li><a href="#about">关于</a></li>
</ul>

加入修饰

    <style>
        .daohang {
        list-style-type:none;//去掉小黑点
        margin:0;//边距设为0
        padding:0;//填充设为0
        }
    </style>

<style>
    .daohang {
    border:1px solid #555;
    list-style-type:none;
    margin:0;
    padding:0;
    background-color:#f1f1f1;
    height:50%;
    /* position:fixed; */
    overflow:auto
    }
    .daohang li {
        border-bottom:1px solid #555;
    }
    .daohang li:last-child{
        border-bottom: none;
    }
    .daohang a{
        text-align:center;
        display:block;
        color: #000;
        padding:8px 16px;
        text-decoration:none;
    }
    .daohang a:hover:not(.active) {
    background-color: #555;
    color: white;
    }
    ul a.active {
        background-color: #4CAF50;
        color: white;
    }
</style>
<ul class="daohang">
  <li><a class="active" href="#home">主页</a></li>
  <li><a href="#news">新闻</a></li>
  <li><a href="#contact">联系</a></li>
  <li><a href="#about">关于</a></li>
</ul>

# css图片廊

    <style>
        div.img {
            margin: 5px;
            border: 5px solid #ccc;
            float: center;
            width: 180px;
        }
        div.img:hover {
            border: 1px solid #777;
        }
        div.desc {
            padding: 15px;
            text-align: auto;
        }
    </style>
    <div>
        <div class="img">
        <a target="_blank" href="https://static.runoob.com/images/demo/demo1.jpg">
        <img src="https://static.runoob.com/images/demo/demo1.jpg" alt="图片描述">
        </a> 
        <div class="desc">
        图文描述
        </div>
        </div>
    </div>

<body>
<style>
    div.img {
        margin: 5px;
        border: 5px solid #ccc;
        float: center;
        width: 180px;
    }
    div.img:hover {
        border: 1px solid #777;
    }
    div.desc {
        padding: 15px;
        text-align: auto;
    }
</style>
<div>
    <div class="img">
    <a target="_blank" href="https://static.runoob.com/images/demo/demo1.jpg">
    <img src="https://static.runoob.com/images/demo/demo1.jpg" alt="图片描述">
    </a> 
    <div class="desc">
    图文描述
    </div>
    </div>
</div>
</body>

