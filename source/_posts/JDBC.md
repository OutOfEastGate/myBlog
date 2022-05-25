---
title: JDBC java数据库连接
date: 2022-03-25 10:24:25
tags: java
---
## 下载安装驱动
[MYSQL驱动jar包下载](https://dev.mysql.com/downloads/connector/j/)
## 将jar包导入idea
在项目下创建新文件夹，将jar包复制进去，右键jar包，选择add as library
## 开始操作数据库
    public static void main(String[] args) throws Exception {
            //注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //获取连接,依次输入ip,端口号和连接的数据库名称
            String url="jdbc:mysql://127.0.0.1:3306/student";
            String username="root";
            String password="password";
            Connection conn=DriverManager.getConnection(url,username,password);
            //定义sql
            String sql="UPDATE stu set math=88 WHERE stuname=\"王洪涛\"";
            //获取执行sql对象的statement
            Statement stmt = conn.createStatement();
            //执行sql
            int count = stmt.executeUpdate(sql);//受影响的行数
            //处理结果
            System.out.println(count);
            //关闭资源
            stmt.close();
            conn.close();
        }
## JDBC API
### DriverManager
获取数据库连接

    .getConnection(url,username,password)

### Connection
1.获取执行SQL的对象

    conn.createStatement();
2.管理事务

    try {
            //开启事务
            conn.setAutoCommit(false);
    
            //执行完毕，提交事务
            conn.commit();
    }catch (SQLException throwables) {
            //如果出现异常,回滚事务
            conn.rollback();
            throwables.printStackTrace();
        }
### statement 执行sql语句
执行sql语句

    .executeUpdate(sql);//更新数据库
    .executeQuery(sql1);//查询数据库
### ResultSet
resultset可以对查询结果进行操作，比如获取查询结果中的各种字段

            Statement stmt = conn.createStatement();
            //执行sql
            ResultSet rs = stmt.executeQuery(sql1);
            re.getInt(列数);//获取相应列的int类型数据
            re.getString(列数);//获取string类型的数据
### PreparedStatement