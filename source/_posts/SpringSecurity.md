---
title: SpringSecurity
date: 2022-07-09 19:07:42
tags: 安全框架
category: 学习笔记-框架
cover: https://images.pexels.com/photos/12499889/pexels-photo-12499889.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2
---

# SpringSecurity在SpringBoot中的应用

## 配置类

```java
@Configuration
@EnableGlobalMethodSecurity(securedEnabled = true) //开启注解后可以在controller方法上进行认证授权
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private DataSource dataSource;//注入数据源
    @Bean
    public PersistentTokenRepository persistentTokenRepository(){
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        return jdbcTokenRepository;
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(password());
    }
    @Bean
    PasswordEncoder password(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling().accessDeniedPage("/unauth.html"); //配置没有权限访问跳转自定义页面
        http.formLogin()
                .loginPage("/login.html") //自定义登录页面
                .loginProcessingUrl("/user/login") //登录访问路径
                .defaultSuccessUrl("/success.html").permitAll() //登录成功后跳转路径
                .and().authorizeRequests()
                    .antMatchers("/","/test/hello","user/login").permitAll() //设置哪些路径可以直接访问
                    .antMatchers("/test/index").hasRole("admin")
                .anyRequest().authenticated()
                .and().rememberMe().tokenRepository(persistentTokenRepository()) //记住我实现单点登录
                .tokenValiditySeconds(60) //有效时间/秒
                .and().csrf().disable();// 关闭csrf防护

        http.logout()
                .logoutSuccessUrl("/test/hello").permitAll();//用户退出后跳转的页面
    }
}
```



测试方法

```java
@Service("userDetailsService")
public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        System.out.println(s);
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_admin");//用户身份/权限
        return new User("wht",new BCryptPasswordEncoder().encode("123"),auths);
    }
}
```

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @GetMapping("hello")
    public String hello(){
        return "Hello SpringSecurtiy";
    }

    @GetMapping("index")
    public String index(){
        return "通过验证";
    }

    @GetMapping("update")   //测试通过注解权限认证
    @Secured("ROLE_admin")     //secured认证角色
//    @PreAuthorize("hasAuthority('admin')") //认证权限
//    @PostAuthorize("hasAuthority('admin')") //方法执行之后在进行验证
    public String update(){
        return "验证通过-管理员身份";
    }
}
```

> SpringSecurity会进行自动认证，所以在前端的form表单中设置action="/user/login"，其中action中的内容要与SpringSecurity配置类中的.loginProcessingUrl一致
>
> SpringSecurity也会自动进行单点登录的验证，也就是表单中的“记住我”选项的name要指定为“remember-me”，SpringSecurity会自动将token值进行和数据库的比对







