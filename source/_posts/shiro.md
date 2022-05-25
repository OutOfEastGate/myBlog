---
title: shiro
date: 2022-04-27 21:25:52
tags: 安全框架
category: 安全框架
cover: https://th.wallhaven.cc/small/8o/8oev1j.jpg
---
![](/img/drop-db.gif)

   **Shiro包含**

Subject 用户

SecurityManager 管理所有用户

Realm 连接数据
## 第一步 引入依赖
```xml
<!--        shiro整合spring的包-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.4.1</version>
        </dependency>
```
## 第二步 写配置shiro的类

认证和授权
```java
public class UserRealm extends AuthorizingRealm {
    @Autowired
    UserService userService;
    @Autowired
    UserMapper userMapper;
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权方法=>"+"doGetAuthorizationInfo");
        //simpleAuthorizationInfo
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //simpleAuthorizationInfo.addStringPermission("user:update");
        //获取当前登录对象
        Subject subject = SecurityUtils.getSubject();
        User currentUser = (User) subject.getPrincipal();
        //查询当前用户是否有权限
        simpleAuthorizationInfo.addStringPermission(currentUser.getPerm());
        return simpleAuthorizationInfo;
    }
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证方法=>"+"doGetAuthenticationInfo");
        //用户名和密码
        String username = "root";
        String password = "123456";
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username","zhangsan");
        List<User> users = userMapper.selectList(queryWrapper);
        User user = users.get(0);
        username=user.getUsername();
        password=user.getPassword();
        UsernamePasswordToken userToken = (UsernamePasswordToken) authenticationToken;
        if(!userToken.getUsername().equals(username)){
            return null;//会自动抛出异常
        }
        //密码认证shiro会自动做
        //可以加密：MD5 MD5盐值加密
        return new SimpleAuthenticationInfo(user,password,"");
    }
}

```
配置
```java
@Configuration
public class ShiroConfig {
    //ShiroFilterFactoryBean第三步

    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //关联defaultWebSecurityManager，设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);
        /*
        anno:无需认证就可以访问
        authc:必须认证了才能访问
        user：必须拥有记住我才能访问
        perms：拥有对某个资源的权限才能访问
        role：拥有某个角色属性才能访问
         */
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<>();
        filterChainDefinitionMap.put("/user/add","perms[user:add]");
        filterChainDefinitionMap.put("/user/update","perms[user:update]");
        //filterChainDefinitionMap.put("/user/*","authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        //设置登录的请求
        shiroFilterFactoryBean.setLoginUrl("/toLogin");
        shiroFilterFactoryBean.setUnauthorizedUrl("/unAuthor");
        return shiroFilterFactoryBean;
    }



    //DefaultWebSecurityManager  第二步

    @Bean(name="securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        //关联UserRealm
        defaultWebSecurityManager.setRealm(userRealm);
        return defaultWebSecurityManager;
    }
    //创建Realm对象 需要自定义  第一步

    @Bean
    public  UserRealm userRealm(){
        return new UserRealm();
    }
}
```
