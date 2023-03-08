---
title: Spring5
date: 2022-04-11 09:11:02
tags: spring
category: 学习笔记-框架
---
# IOC
## 什么是IOC
  - 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理
  - 使用IOC的目的：降低耦合度
---

## IOC 底层原理
  - xml解析，工厂模式，反射
## IOC过程

第一步 xml配置文件，配置创建的对象

        <bean id="user" class="com.wht.User"></bean>
第二部 有service类和dao类，创建工厂类

    public static UserDao getDao(){
        String classValue = class属性值;//通过xml解析
        Class class = Class.forNmae(classValue);//反射
        return (UserDao)class.newInstance();
    }
---
## IOC(接口)
  - IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
  - Spring提供IOC容器实现的两种方式：（两个接口）
  - BeanFactory：IOC容器基本实现，是Spring内部使用的接口，不提供开发人员使用，加载配置文件时不会创建对象，在获取对象时才去创建对象
  - ApplicationContext：BeanFactory接口的子接口，提供更多功能，开发人员使用，加载配置文件时就会把配置文件中的对象创建
  - 第二种更好，因为在可以做到慢启动，快响应
---
## IOC 操作 Bean管理
  - Bean管理是指两个操作
  - Spring创建对象
  - Spring注入属性
  ### Bean 管理操作的两种方式
- **xml创建对象**
    ![](img/../../../public/img/../../themes/volantis/source/img/012.png)
  
  默认执行无参构造函数

  id属性：唯一标识

  class属性：类全路径

- **xml 注入属性**

1.使用set方法

2.使用有参构造

DI：IOC的一种方法，依赖注入，就是注入属性

第一种注入方式：使用set方法

![](img/../../../themes/volantis/source/img/Spring-test1.png)

第二种注入方式：使用有参构造
![](img/../../../themes/volantis/source/img/Spring-test2.png)
****
- 注入空值

        <constructor-arg name="bName">
                    <null></null>
        </constructor-arg>
- 注入特殊字符

1.可以使用转义

2.使用CDATA

      <constructor-arg name="bName">
                <value> <![CDATA[<<数据结构>>]]> </value>
              </constructor-arg>
- 注入外部bean

        <!--    注入外部bean-->
            <property name="userDao" ref="userDaoImpl"></property>
        </bean>
        <bean id="userDaoImpl" class="com.wht.dao.UserDaoImpl"></bean>
- 注入内部bean(实现一对多的关系) 也叫级联赋值

比如一个员工属于一个部门，但一个部门有多个员工，可以在员工类内加上一个部门对象，通过给部门这个属性赋值来规定员工的部门，这时可以用Spring来实现这种关系

    <!--注入内部bean-->
        <bean id="Emp" class="com.wht.pojo.Emp">
            <property name="name" value="wht"></property>
            <property name="dept" ref="Dept"></property></bean>
        <bean id="Dept" class="com.wht.pojo.Dept">
            <property name="dName" value="安保部门"></property>
        </bean>

----
- 注入数组类型的属性

        <property name="strings">
                    <array>
                        <value>数组1</value>
                        <value>数组2</value>
                    </array>
                </property>

- 注入List集合类型属性

        <property name="list">
                    <list>
                        <value>list1</value>
                        <value>list2</value>
                    </list>
                </property>

- 注入Map集合类型的属性

        <property name="map">
                    <map>
                        <entry key="key1" value="value1"></entry>
                        <entry key="key2" value="value2"></entry>
                    </map>
                </property>
- 注入Set集合类型的属性

        <property name="set">
            <set>
                <value>set1</value>
                <value>set2</value>
            </set>
        </property>

- set类型：
   - set集合不允许有重复的如果内容重复，就只存进去第一次的，后面的就存不进去
   - set类型集合没有索引
- 如果注入的list集合类型是对象，则把value标签换成ref标签即可

----
- 提取list集合类型属性注入

第一步 在xml中使用util
![](img/../../../themes/volantis/source/img/Spring-test3.png)

第二步 提取list集合

    <util:list id="booklist">
        <value>list-1</value>
        <value>list-2</value>
        <value>list-3</value>
    </util:list>
第三步 注入list集合

    <bean id="book" class="com.wht.pojo.Book">
            <property name="bAuthor" value="author"></property>
            <property name="bName" value="book"></property>
            <property name="booklist" ref="booklist"></property>
        </bean>
----
## IOC操作Bean管理（FactoryBean）
Spring有两种bean，一种普通bean，另一种工厂bean（FactoryBean）

- 普通bean：在配置文件中定义的bean类型就是返回类型
- 工厂bean：在配置文件定义bean类型可以和返回值不一样
----
## IOC操作Bean管理（bran作用域）
在Spring中，默认情况下，bean时单实例对象，也就是多次创建都是同一个对象

可以设置成多实例 使用scope设置

- scope属性值
  - 第一个值 singleton 单实例
  - 第二个值 prototype 多实例
- 区别
  - 单实例在加载配置文件时完成对象创建
  - 不是在加载Spring对象时调用，而是在调用时才创建，每次创建的都是不同的对象

scope还有不常用的属性request和session

----
## IOC操作Bean管理(bean生命周期)
- bean生命周期
  
  1.通过构造器创建bean实例（默认无参构造）
  
  2.为bean的属性设置值和对其他bean引用（调用set方法）

  3.调用bean初始化的方法（需要进行配置）

  4.bean可以使用

  5.当容器关闭时，调用bean销毁方法（需要配置销毁的方法）

配置bean初始化的方法 init-method

手动销毁bean context.close() 会自动强转

配置bean销毁的方法 destory-method

- bean后置处理器
  
  配置后置处理器后，会在第三步，调用bean初始化之前和调用bean初始化之后分别执行
  - 把bean实例传递给后置处理器的方法
  - 把bean实例传递给后置处理器后的方法

- bean配置后置处理器的方法
  
  在对应的对象类中实现BeanPostProducer接口并实现其两个方法

        @Override
      public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
          System.out.println("1.bean传递给后置处理器");
          return bean;
      }
      
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          System.out.println("2.bean传递给后置处理器");
          return bean;
      }

----
## IOC操作Bean管理（xml自动装配）
- byName自动装配

         <bean id="emp" class="com.wht.autowire.Emp" autowire="byName"></bean>
     ```
       <bean id="dept" class="com.wht.autowire.Dept">
           <property name="dName" value="划水部门"></property>
       </bean>
     ```

     通过name属性自动装配，其中要装配的对象（划水部门）的name必须和bean内对应的属性name一致才可以

![](img/../../../themes/volantis/source/img/Spring-test4.png)

- 根据属性内容装配byType

## IOC操作Bean管理（引入外部属性文件）
- 普通连接方式

        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
            <property name="url" value="jdbc:mysql://localhost:3306/users"></property>
            <property name="username" value="root"></property>
            <property name="password" value="password"></property>
        </bean>
- 使用外部属性文件进行连接的方式

第一步：在src下创建properties文件

第二步:在properties文件中写入信息

    prop.driverClass=com.mysql.jdbc.Driver
    prop.url=jdbc:mysql://localhost:3306/users
    prop.username=root
    prop.password=password
第三步：在xml文件中引入context
![](img/../../../themes/volantis/source/img/Spring-test5.png)
第四步：引入外部属性文件

    <!--引入外部属性文件-->
        <context:property-placeholder location="jdbc.properties"></context:property-placeholder>
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${prop.driverClass}"></property>
            <property name="url" value="jdbc:mysql:${prop.url}"></property>
            <property name="username" value="${prop.username}"></property>
            <property name="password" value="${prop.password}"></property>
        </bean>

----
## IOC操作Bean管理(基于注解方式)
### Spring针对Bean管理中创建对象提供的注解
- @Component
- @Service
- @Controller
- @Repository

四个注解的功能都是一样的，都可以创建bean实例
### 基于注解方式实现对象创建
第一步：引入依赖

第二步：开启组件扫描

      <context:component-scan base-package="com.wht"></context:component-scan>

第三步：开始使用注解创建对象

![](img/../../../themes/volantis/source/img/Spring-test5.png)

在xml中可以配置哪些扫描哪些不扫描

----
## 注解方式注入属性
- @AutoWired 根据属性类型进行输入
- @Qualifier 根据属性名称注入
- @Resource 可以根据类型注入，也可以根据名称注入

使用Qualifier必须和AutoWired一起使用，在一个接口有多个实现类的时候可以进行使用

      //使用注解进行注入不需要设置set方法
      @Autowired //根据类型进行注入
      @Qualifier(value = "userDaoImpl")//如果一个接口有多个实现类，需要用名称
      private UserDao userDao;

使用Resource注入

    @Resource//根据类型
    @Resource(name = "userDaoImpl")//根据名称

使用Value注入

    @Value(value = "test1")
    private  String test;

### 完全注解开发
首先创建一个config类，在类中加入如下注解

    @Configuration//替代配置文件
    @ComponentScan(basePackages = "com.wht")

然后在测试类中把加载配置文件换成加载配置类

    ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);

----
# AOP
## 什么是AOP
AOP是面向切面（方面）编程的缩写降低业务逻辑的耦合度，提高程序的可重用性，同时提高开发的效率

通俗理解：不通过修改源代码来增加一些新的功能

---
## AOP底层原理
- AOP底层使用动态代理
  - 有两种情况
    - 第一种：有接口
      
      使用JDK动态代理，创建接口的代理对象，来增加一些新的功能
    - 第二种：没有接口

      CGLIB动态代理，创建子类的代理对象，增强类中的方法
---
## AOP术语
- 连接点

  类里面哪些方法可以被增强，这些方法称为连接点
- 切入点

  实际真正增强的方法称为切入点
- 通知（增强）

  实际增强的逻辑部分就叫通知

  通知有多种类型
  - 前置
  - 后置
  - 环绕
  - 异常
  - 最终
- 切面

  是一个动作，把通知应用到切入点的过程就叫切面
----
## AOP操作
- Spring框架一般基于AspectJ实现AOP操作
- AspectJ本身是一个单独的框架，一般把AspectJ和Spring框架一起使用，进行AOP操作
### 基于AspectJ实现AOP操作
- 基于xml配置文件实现

- 基于注解方式实现（使用较多）

第一步 导入依赖

    <!--        Aop相关依赖-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
                <version>5.2.6.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjweaver</artifactId>
                <version>1.9.3</version>
                <--注意版本问题-->
            </dependency>
            <dependency>
                <groupId>aopalliance</groupId>
                <artifactId>aopalliance</artifactId>
                <version>1.0</version>
            </dependency>
            <dependency>
                <groupId>net.sourceforge.cglib</groupId>
                <artifactId>com.springsource.net.sf.cglib</artifactId>
                <version>2.2.0</version>
            </dependency>
- 切入点表达式
语法结构

execution([权限修饰符][返回类型][类全路径][方法名称][参数列表])

例如 对Book类中的add方法进行修改

    execution(* com.wht.dao.Book.add(..))
对dao包里面所有类，类里面的所有方法进行增强

execution(* com.wht.dao.*.*(..))

---
## AOP操作（AspectJ注解）
第一步：开启注解扫描和Aspect

            <!--开启注解扫描-->
            <context:component-scan base-package="com.wht"></context:component-scan>
            <!--    开启Aspect生成代理对象-->
            <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

第二部：在增强类和被增强类中分别加入注解
![](/img/Aop-1.png)
![](img/Aop-2.png)
**其他通知**

      //前置通知
      @Before(value = "execution(* com.wht.pojo.User.add(..))")
      public void before(){
          System.out.println("before...");
      }
      //最终通知
      @After(value = "execution(* com.wht.pojo.User.add(..))")
      public void after(){
          System.out.println("after...");
      }
      //异常通知，当有异常时执行
      @AfterThrowing(value = "execution(* com.wht.pojo.User.add(..))")
      public void afterThrowing(){
          System.out.println("afterThrowing...");
      }
      //后置通知
      @AfterReturning(value = "execution(* com.wht.pojo.User.add(..))")
      public void afterReturning(){
          System.out.println("afterReturning...");
      }
      //环绕通知，在被增强方法之前和之后执行
      @Around(value = "execution(* com.wht.pojo.User.add(..))")
      public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
          System.out.println("around before");
          proceedingJoinPoint.proceed();
          System.out.println("around after");
      }
- after是最终通知，不管有没有异常都会执行
- AfterThrowing是异常通知，当有异常时执行
- 当有异常是，环绕通知和后置通知(afterReturn)都不会执行
### **相同切入点抽取**
因为每个通知后面的切入点都是一样的，所以可以进行相同切入点抽取

    @Pointcut(value = "execution(* com.wht.pojo.User.add(..))")
    public void pointcut(){
        //相同切入点抽取
    }
    //前置通知
    @Before(value = "pointcut()")
    public void before(){
        System.out.println("before...");
    }
### 多个增强类对同一个方法进行增强
当多个增强类对同一个方法进行增强时，可以设置增强类优先级

    //增强的类
    @Component
    @Aspect //生成代理对象
    @Order(1)//设置优先级，值越小，越优先执行

----
## AOP操作（AspectJ配置文件）

    <!--    创建对象-->
        <bean id="user" class="com.wht.pojo.User"></bean>
        <bean id="userProxy" class="com.wht.aopxml.UserProxy"></bean>
    <aop:config>
    <!--    切入点-->
        <aop:pointcut id="p" expression="execution(* com.wht.pojo.User.add(..))"/>
    <!--配置切面-->
        <aop:aspect ref="userProxy">
            <aop:before method="before" pointcut-ref="p"></aop:before>
        </aop:aspect>
    </aop:config>
### AOP全注解开发
创建一个配置类，不需要xml文件

    @Configuration//配置类
    @ComponentScan(basePackages = "com.wht")//开启注解扫描
    @EnableAspectJAutoProxy(proxyTargetClass = true)//开启Aspect生成代理对象
    public class ConfigAop {
        
    }
----
# JdbcTemplate
## 什么是JdbcTemplate
Spring框架对JDBC进行了封装，使用JdbcTemplate对数据库进行操作

----
## JdbcTemple操作数据库
第一步 ： 配置xml文件

    <!--开启注解扫描-->
        <context:component-scan base-package="com.wht"></context:component-scan>
    
    <!--引入外部属性文件-->
        <context:property-placeholder location="jdbc.properties"></context:property-placeholder>
    <!--数据库连接池-->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${prop.driverClass}"></property>
            <property name="url" value="${prop.url}"></property>
            <property name="username" value="${prop.username}"></property>
            <property name="password" value="${prop.password}"></property>
        </bean>
    <!--    JdbcTemplate对象-->
        <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!--        注入dataSource-->
            <property name="dataSource" ref="dataSource"></property>
        </bean>
第二步： 添加注解

在userdao的实现方法中

      @Repository
      public class UserDaoImpl implements UserDao{
          //注入JdbcTemplate
          @Autowired
          private JdbcTemplate jdbcTemplate;
          @Override
      public void add(User user) {
          String sql="insert into users values(?,?)";
          int update = jdbcTemplate.update(sql, user.getPassword(), user.getUsername());
          System.out.println(update);
      }
    }
在UserService中

    @Service(value = "userService") 
    public class UserService {
          @Autowired
        private UserDao userDao;
        public void add(User user){
            userDao.add(user);
        }
    }
第三步：在Test类中调用

---
# 事务
## 什么是事务
事务时数据库操作的最基本单元，逻辑上的一组操作，要么都成功，要么都失败
- 事务的四个特性
  - 原子性（不可分割）
  - 一致性
  - 隔离性
  - 持久性
  
        //事务操作
          public void account() {
              try {
                  //开启事务
                  //没有出现异常,提交事务
              } catch (Exception e) {
                  //出现异常,事务回滚
              }
          }//account
----
## 事务操作（Spring事务管理介绍）
- 事务一般添加在JavaEE三层结构里面的Service层(业务逻辑层)
- 在Spring中进行事务操作有两种方式
  - 编程式
  - 声明式（使用较多）
- 在Spring中进行声明式事务管理，底层使用AOP原理
----
## Spring事务管理API
Spring提供一个接口，代表事务管理器，这个接口针对不同框架提供不同的实现类

----
## 在Spring配置文件配置事务管理器
第一步：配置事务管理器

    <!--    配置事务管理器-->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
第二步：注入数据源，并开启事务管理

    <!--        注入数据源-->
            <property name="dataSource" ref="dataSource"></property>
        </bean>
    <!--    开启事务管理-->
        <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
第三步：添加注解

- @Transactional，这个注解可以添加到类上面，也可以添加到方法上面
- 注解添加到类上面，这个类里面的所有方法都添加了事务
- 注解添加到方法上，为这个方法添加事务
### Transactional中配置事务相关参数
- propagation：事务传播行为 

默认 @Transactional(propagation = Propagation.REQUIRED)
也就是在其他方法中如果没有事务，会自动添加事务
- ioslation：事务隔离级别
  - 事务有隔离性的特性，多事务操作之间不会产生影响，不考虑隔离性产生很多问题
  - 有三个问题：脏读，不可重复读，虚读（幻读）
  - 脏读：一个未提交事务读取到另一个未提交事务的数据
  - 不可重复读：一个未提交事务读取到另一个已经提交的事务的数据（是一种现象而不是一个问题） 
  - 虚读：

一个未提交事务读取到另一提交事务添加数据
- timeout：超时时间

  - 事务需要在一定时间内提交，否则回滚

  - timeout默认是-1，可以设置以秒为单位
- readOnly：是否只读

  - 默认值是false，可以查询也可以添加修改删除操作
- rollBackFor：回滚
  - 设置出现哪些异常进行事务回滚
- noRollbackFor：不回滚
  - 设置出现哪些异常不进行回滚
----
## 事务操作（XML声明式事务管理）
第一步：配置事务管理器

第二步：配置通知

第三步：配置切入点和切面

    <!-- 1.   配置事务管理器-->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--        注入数据源-->
            <property name="dataSource" ref="dataSource"></property>
        </bean>
    <!--2.配置通知-->
        <tx:advice id="txadvice">
            <tx:attributes>
    <!--            指定那种规则的方法上面添加事务-->
                <tx:method name="accountMoney" propagation="REQUIRED"/><!--指定accountMoney方法-->
                <tx:method name="account*"/> <!--指定以account开头的方法-->
            </tx:attributes>
        </tx:advice>
            <!--    3.配置切入点和切入面-->
        <aop:config>
    <!--        配置切入点-->
            <aop:pointcut id="pt" expression="execution(* com.wht.service.UserService.add(..))"/>
    <!--        配置切面-->
            <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
        </aop:config>
----
## 事务操作（完全注解声明式事务管理）
第一步：创建配置类

    @Configuration//配置类
    @ComponentScan(basePackages = "com.wht")//组件扫描
    @EnableTransactionManagement//开启事务
    public class TxConfig {
        //创建数据库连接池
        @Bean
        public DruidDataSource getDruidDataSource(){
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setDriverClassName("com.mysql.jdbc.Driver");
            dataSource.setUrl("jdbc:mysql://localhost:3306/student");
            dataSource.setUsername("root");
            dataSource.setPassword("password");
            return dataSource;
        }
        //创建JdbcTemplate对象
        @Bean
        public JdbcTemplate jdbcTemplate(DataSource dataSource){
            JdbcTemplate jdbcTemplate = new JdbcTemplate();
            //注入datasource
            jdbcTemplate.setDataSource(dataSource);
            return jdbcTemplate;
        }
        //创建事务管理器
        @Bean
        public DataSourceTransactionManager dataSourceTransactionManager(DataSource dataSource){
            DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
            dataSourceTransactionManager.setDataSource(dataSource);
            return dataSourceTransactionManager;
        }
    }

----
## Spring 5新特性
- 基于jdk8
- 日志
- 新增@nullable注解，可以使方法返回值为空，也可以使用在方法参数中和属性值中
- 函数式风格Lambda表达式
----
## SpringWebflux介绍
2022/4/13

