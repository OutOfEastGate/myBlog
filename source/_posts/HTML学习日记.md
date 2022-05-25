---
title: HTML学习日记
date: 2022-03-19 09:12:51
tags: 前端
category: 前端
---

# HTML简介
HTML是超文本标记语言(Hyper Text Markup Language)，是用来描述网页的一种语言
HTML不是编程语言，而是一种标记语言，标记语言是一套标记标签
---

# Web标准
## Web标准的构成
标准|说明
-|-|
结构|用于对网页元素进行整理和分类，主要是HTML
表现|用于设置网页元素的板式、颜色、大小等外观，主要是css
行为|指网页模型的定义及交互的编写，主要是JavaScript
---
# HTML语法规范
## 基本语法
+ HTML一般成对出现,有单标签，但是很少
## 标签关系
- 包含关系
```html
    <head>
    < title> < /title>
    </head>
```
- 并列关系
```html
    <head> </head>
    <body> </body>
```
## HTML基本结构标签


标签名|定义|说明
-|-|-|
```<html></html>```|HTML标签|页面最大标签，也称根标签
```<head></head>```|文档的头部|在head中必须设置title标签
```<title></title>```|文档的标题|让页面有自己的标题
```<body></body>```|文档的主体|页面内容基本都是在body中

### 文档类型声明标签
```<!DOCTYPE html>```意思是告诉浏览器使用了那种HTML版本来显示网页
这个标签表示使用最新的HTML5显示页面
### lang语言类型
用来定义当前文档显示的语言，en为英语，zh-CN定义为中文网页
不影响页面显示，只是对浏览器提示
### 字符集
在```<head>```标签中，可以通过```<meta>```标签的charset属性来规定HTML文档应该使用哪种字符编码,例如
```<meta charset="UTF-8">```

## HTML常用标签
### 标题标签```<h1>-<h6>```
```<h1>```是一级标签,以此类推
特点：
- 加了标题的文字会变大变粗
- 每个标题都是独占一行

### 段落标签和换行标签
在HTML中，多个空格或者多个回车并不会起作用，必须使用标签
```<p></p>```是一个段落标签（paragraph的缩写）
特点：
- 文本会根据浏览器窗口大小自动换行
- 段落和段落间会有空隙
```<br />```是一个换行标签（break的缩写）
特点：
- 这是一个单标签
- 会另起一行但不会有上下间距

### 文本格式化标签
设置文字<strong>粗体</strong>、<em>斜体</em>或<ins>下划线</ins>等效果

语义|标签|说明
-|-|-|
加粗|```<strong></strong>或<b></b>```|推荐用```<strong>```
倾斜|```<em></em>或<i></i>```|推荐使用```<em>```
删除线|```<del></del>或<s></s>```|推荐```<del>```
下划线|```<ins></ins>或<u></u>```|推荐```<ins></ins>```

### ```<div>和<span>```标签
```<div>和<span>```是没有语义的，它们就是一个盒子，用来装内容的

特点：
- div是division的缩写(分割，分区),单独占一行，用作大盒子
- span(跨度、跨距)，一行可以放好多个，用作小盒子

### 图像标签和路径

### 图像标签
在HTML标签中，```<img>```标签用于定义HTML页面中的图像
```<img src="狐狸.jpg"/>```
图像标签的其他属性

属性|属性值|说明
-|-|-|
src|图片路径|必须属性
alt|文本|替换文本，图像不能正常显示，用该文字替代
title|文本|提示文本，鼠标放到图像上的提示文字
width|像素|设置图像的宽度
height|像素|设置图像的高度
border|像素|设置图像的边框粗细

例如：
```<img src="img.png" title="图像"/>```属性之间不分顺序

### 路径
#### 相对路径
相对路径分类|符号|说明
-|-|-|
同一级路径| |图像与html文件处于同一级如```<img src="1.png"/>```
下一级路径|/|图像在HTML文件的下一级如```<img src="/img/1.png"/>```
上一级路径|../|图像在html文件的上一级```<img src="../1.png/>```

#### 绝对路径
完整的路径或网络地址
```"C:\Users\1.png"或"www.xxx.com"```

### 超链接标签

- a是anchor（锚）的缩写，herf是网站地址，是必须属性，target是打开方式，可选"_blank"或"_self",标签中间可以是文字或者图片

- 链接可以分为内部链接和外部链接

- 下载链接:herf地址是一个文件或压缩包，会下载这个文件

- 空链接：herf里是#即可

```<a href="https://home.firefoxchina.cn" target="_blank"> 火狐</a>```

<a href="https://home.firefoxchina.cn/" target="_blank"> 火狐</a>

```<a href="http://www.qq.com" target="_self"> 腾讯</a>```

<a href="http://www.qq.com" target="_self"> 腾讯</a>

### 锚点链接
1.在链接文本的href属性中，设置属性值为#名字的形式,如：

```<a href="#two"> 第二季</a>```

2.在目标位置添加id属性

```<h3 id="two"> 第二季</a>```

### 注释和特殊字符
```<!--注释文字--> 快捷键ctrl + /```

![](/img/zifu.jpg)

## 表格标签
- ```<table></table>```是定义表格的标签
- ```<td></td>```是定义表格中的行的标签
- ```<td></td>```定义表格中行的单元格



       <table>
       <tr><td> 年龄</td> <td>性别 </td> <td>姓名</td></tr>
       <tr><td>19</td> <td>男</td><td> 王洪涛</td></tr>
       </table>


效果：
<table>
    <tr><td> 年龄</td> <td>性别 </td> <td>姓名</td></tr>
    <tr><td>19</td> <td>男</td><td> 王洪涛</td></tr>
</table>

### 表头单元格
```<th></th>```

在表头加入表头标签会使表头文字加粗并居中，效果如下
<table>
<tr><th>年龄</th> <th>性别</th> <th>姓名</th></tr>
<tr><td>19</td> <td>男</td><td> 王洪涛</td></tr>
</table>

### 表格属性
不建议在html中使用，一般在css中使用

属性名|属性值|描述
-|-|-|
align|left、center、right|规定表格在网页中的对齐方式
border|1或""|规定表格单元是否拥有边框，默认没有，1表示有
cellpadding|像素值|规定单元边沿与其内容之间的空白，默认为1
cellspacing|像素值|规定单元格之间的空白，默认为2
width|像素值或百分比|规定表格的宽度
height|像素|规定表格的高度

### 表格结构标签
```<thead></thead> <tbody></tbody>```

thead内必须有tr
### 合并单元格
- 跨行合并 rowspan=“合并单元格的个数”，最上侧单元格为目标单元格，写合并代码
- 跨列合并 colspan=“合并单元格的个数”，最左侧单元格为目标单元格，写合并代码


        <table width="500" height="249" border="1" cellspacing="0">
        <tr>
        <td> </td><td colspan=2> </td>
        </tr>
        <tr>
        <td> </td><td> </td><td> </td>
        </tr>
        <tr>
        <td> </td><td> </td><td> </td>
        </tr>
        </table>
效果如下：
<table border="1" cellspacing="0">
    <tr>
    <td> </td><td colspan=2> </td>
    </tr>
    <tr rowspan= 2>
    <td> </td><td> </td><td> </td>
    </tr>
    <tr>
    <td> </td><td> </td><td> </td>
    </tr>
</table>

## 列表标签 
### 无序列表
    <ul>
    <li>1</li>
    <li>2</li>
    </ul>
效果如下
<ul>
<li>1</li>
<li>2</li>
</ul>
注意：ul中只能放li标签，但li里面可以放任意标签

### 有序列表
    <ol>
    <li>1</li>
    <li>2</li>
    </ol>
效果如下
<ol>
<li>第一</li>
<li>第二</li>
</ol>

### 自定义列表
    <dl>
        <dt>联系方式</dt>
        <dd>QQ</dd>
        <dd>微信</dd>
    </dl>
效果如下：
<dl>
    <dt>联系方式</dt>
    <dd>QQ</dd>
    <dd>微信</dd>
</dl>

## 表单标签
### 表单的组成
在HTML中，一个完整的表单由三部分组成，分别是
<ul>
<li>表单域</li>
<li>表单控件</li>
<li>提示信息</li>
</ul>

### 表单域
在HTML标签中，```<form></form>```标签用于定义表单域
form会把它范围内的表单元素提交给服务器

    <form action="url地址" method= "提交方式" name="表单域名称">
    各种表单元素控件
    </form>

常用属性：

属性|属性值|作用
-|-|-|
action|url地址|用于指定接收并处理表单数据的服务器程序的url地址
method|get/post|用于设置表单数据的提交方式，取其值为get或post
name|名称|用于指定表单的名称，以区分同一个页面中的多个表单域

### 表单元素
#### ```<input>```表单元素
```<input type="属性值"/>```<br>

    <form>
    用户名：<input type="text" name="username" value="请输入用户名"/><br>
    密码:<input type="password" name="password">
    性别:男<input type="radio" name="sex">女<input type="radio" name="sex">
    爱好:男<input type="chekbox" name="hobby">女<input type="chekbox" name="hobby">
    <input type="submit">
    <input type="button" value="按钮">
    上传文件:<input type="file">
    </form>
效果如下：<br>
<form>
用户名：<input type="text" name="username" value="请输入用户名"/>

密码:<input type="password" name="password">

性别:男<input type="radio" name="sex">女<input type="radio" name="sex"><br>

选框组件可以选择checked属性为checked，默认勾选

爱好:男<input type="checkbox" name="hobby">女<input type="checkbox" name="hobby" checked="checked">

<input type="submit" value="免费注册">

<input type="reset" value="重新设置">

<input type="button" value="按钮">

上传文件:<input type="file">
</form>

### lable标签
lable标签可以绑定表单组件，使操作更快捷

    <label for="text"> 用户名:</label><input type="checkbox" id="text">
效果如下:

<label for="text"> 用户名:</label><input type="checkbox" id="text">

### 下拉标签
    <select>
        <option selected="selected">男</option>
        <option>女</option>
    </select>
效果：

性别:
<select>
    <option selected="selected">男</option>
    <option>女</option>
</select>

### 文本域标签
    <textarea cols="50" rows="5">输入反馈</textarea>
    cols和rows表示显示的列数和行数
今日反馈:
<textarea cols="50" rows="5">输入反馈</textarea>

