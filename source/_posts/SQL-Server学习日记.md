---
title: SQL Server学习日记
date: 2022-03-17 21:09:29
tags: 学习日记
---
# 创建数据库
（1）	数据库名称 Test2。
（2）	主要数据文件：逻辑文件名为Test2_data1，物理文件名为Test2_data1.mdf，初始容量为10MB，最大容量为100MB，增幅为10MB。
次要数据文件会自动生成

    create database Test2
    on primary (name = Test2_data1,
    filename = 'd:\SQL Server Project\Test2\Test2_data1.mdf',
    size=10,maxsize=100,filegrowth=10)

    log on(name=Test2_log1,
    filename = 'd:\SQL Server Project\Test2\Test2_data2.ndf',
    size=10,maxsize=50,filegrowth=20)

# 修改数据库
## 数据库更名
    alter database test1  modify name=new_test1

## 修改数据库属性

    alter database Test2
    modify file(name=Test2_data1,
    size=50,maxsize=200,filegrowth=20),
    (name=Test2_data2,
    size=50,maxsize=300,filegrowth=20),
    (name=Test2_log1,
    size=30,maxsize=100,filegrowth=10%）

# 创建表格
- 学生表格
  

        create table student(        
        Student_id char(10) not null primary key,  
        Student_name char(10) not null ,
        sex char(1) not null constraint sex check(sex='F' or sex='M') ,
        birth smalldatetime not null,
        age int,
        department char(15) not null default '计算机系')

- 课程表格

        create table course
        (Course_id char(6)not null primary key clustered,
        Course_name char(20) not null ,
        PreCloud char(6),
        credits tinyint not null )

- score 表格
  
        create table score
        (Student_id char(10) references student(Student_id),
        Course_id char(6) references course(Course_id),
        Grade tinyint constraint Grade check(Grade>0 and Grade<100))

# 修改表格
（1）	为表student增加一个memo（备注）字段，类型为varchar（200）。

    alter table student add memo varchar(200)

（2）	将memo字段的数据类型更改为varchar（300）。

    alter table student
    Alter column memo varchar(300)

（3）	删除memo字段

    alter table student drop column memo
（4）   添加记录

    insert into 表格名字 values(数据)
# 删除表格

    drop table course