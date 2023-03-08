---
title: vscode连接使用MySQL
date: 2022-03-18 09:18:02
tags:
---

# 下载MySQL    

## 去官网下载MySQL

---

# 配置vscode
[下载Mysql](https://www.mysql.com/)
[安装配置Mysql](https://blog.csdn.net/weixin_43579015/article/details/117228159?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164756193916780357287774%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164756193916780357287774&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-117228159.142^v2^pc_search_result_control_group,143^v4^register&utm_term=mysql%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B&spm=1018.2226.3001.4187)
## 修改c_cpp_properties
在c_cpp_properties中做出如下修改
![c_cpp_properties](/img/002.png)
## 修改launch
在launch中做出如下修改
![launch](/img/003.png)
## 修改tasks
在tasks中作出如下修改
![tasks](/img/004.png)

---

# 连接MySQL

## 安装MySQL插件
![MySQL插件](/img/005.png)
安装完成后会在左下角出现该图标，点击加号
![MySQL](/img/006.png)
如果找不到该图标可以检查资源管理器旁边的设置并勾选上MySQL
![MySQL](/img/005.1.png)
依次输入localhost(或127.0.0.1)，用户名(默认是root),密码(MySQL设置的密码),端口号一般默认是3306,后面直接回车跳过即可

连接成功之后会出现下图所示，系统会自带几个database
![MYSQL](/img/007.png)
## 使用MySQL
此时可以正式在vscode中使用Mysql了，右击数据库会生成一个.sql文件
![Mysql](/img/008.png)
使用sql语句，右击，选择Run MySQL Query 
[sql语句](https://blog.csdn.net/hzw6991/article/details/87757426?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164760930716780357299211%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164760930716780357299211&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-87757426.142^v2^pc_search_result_control_group,143^v4^register&utm_term=mysql%E8%AF%AD%E5%8F%A5&spm=1018.2226.3001.4187)

![MYSQL](/img/009.png)
运行结果截图
![MySQL](/img/010.png)
---

## 参考文献


