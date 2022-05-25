---
title: jwt
date: 2022-04-28 09:44:59
tags: 安全框架
category: 安全框架
cover: https://w.wallhaven.cc/full/y8/wallhaven-y8lqo7.jpg
---
## JWT介绍
### JWT与shiro
shiro和JWT是典型的有状态登陆和无状态登陆的代表，所谓的有状态和无状态，就是看信息存在哪儿，存在服务器端，就叫做有状态，存在客户端，就叫无状态。

有状态就是说把信息存储在session中，因为session是存在服务端的，也就是有状态的，无状态就是把信息存在cookie中，cookie是存在客户端的，也就是无状态。

无状态的好处很明显，不存在服务端，可以减少服务端的压力
### JWT的组成
JWT由三部分拼接组成，中间用“.”分割

- Header

alg属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）；

typ属性表示令牌的类型，JWT令牌统一写为JWT。最后，使用**Base64 URL**算法将上述JSON对象转换为字符串保存
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
- Payload


```
iss：发行人
exp：到期时间
sub：主题
aud：用户
nbf：在此之前不可用
iat：发布时间
jti：JWT ID用于标识该JWT
```
该部分不可以将密码等敏感信息存入
- Signature
  

签名哈希部分是对上面两部分数据签名，需要使用base64编码后的header和payload数据，通过指定的算法生成哈希，以确保数据不会被篡改。首先，需要指定一个密钥（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用header中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名
$$
HMACSHA256(base64UrlEncode(header)+"."+base64UrlEncode(payload),secret)
$$

### JWT验证逻辑
<img src="https://img-blog.csdnimg.cn/img_convert/900b3e81f832b2f08c2e8aabb540536a.png" style="zoom: 67%;" />

> 原文链接：https://blog.csdn.net/weixin_45070175/article/details/118559272

## JWT使用

### 快速开始

第一步 引入依赖

```xml
<!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
<!--        高版本jdk需要加入以下依赖-->
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.0</version>
        </dependency>
        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-impl</artifactId>
            <version>2.3.0</version>
        </dependency>
        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-core</artifactId>
            <version>2.3.0</version>
        </dependency>
        <dependency>
            <groupId>javax.activation</groupId>
            <artifactId>activation</artifactId>
            <version>1.1.1</version>
        </dependency>
<!--        jjwt在0.10版本以后发生了较大变化，pom依赖要引入多个-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>

```

第二步 测试JWT

```java

@SpringBootTest
class JwtDemoApplicationTests {

    private long time = 1000*60*60*24;//一小时
    private String signature = "admin";
    @Test
    void contextLoads() {
    }

    @Test
    void testJWT() {
        JwtBuilder jwtBuilder= Jwts.builder();
        String jwtToken = jwtBuilder
                //头信息
                .setHeaderParam("typ","JWT")
                .setHeaderParam("alg","HS256")
                //载荷payload
                .claim("username","tom")
                .claim("role","admin")
                .setSubject("admin-test")
                .setExpiration(new Date(System.currentTimeMillis()+time))
                .setId(UUID.randomUUID().toString())
                //signature
                .signWith(SignatureAlgorithm.HS256,signature)
                //拼接
                .compact();
        System.out.println(jwtToken);
    }

    @Test
    public void parse(){
        String token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InRvbSIsInJvbGUiOiJhZG1pbiIsInN1YiI6ImFkbWluLXRlc3QiLCJleHAiOjE2NTExOTg4NDEsImp0aSI6ImI4NzY2Mzc3LTJjNjEtNDg1Mi04MTQzLTNhMzc3OTRlMGM3MyJ9.fqhPYypuPj0W7qdPqXxmcM9xHTiRJMEHteqz4cJPf_c";
        JwtParser jwtParser = Jwts.parser();
        Jws<Claims> claimsJws = jwtParser.setSigningKey(signature).parseClaimsJws(token);
        Claims claims = claimsJws.getBody();
        System.out.println(claims.get("username"));
        System.out.println(claims.get("role"));
        System.out.println(claims.getSubject());
        System.out.println(claims.getExpiration());
    }
}
```

### 在工程中使用

**JWTUtils**

```java
package com.wht.myblogapi.utils;

import io.jsonwebtoken.Jwt;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JWTUtils {

    private static final String jwtToken = "123456Mszlu!@#$$";
    private static long time = 1000*60*60*24;//一小时
    public static String createToken(Long userId){
        Map<String,Object> claims = new HashMap<>();
        claims.put("userId",userId);
        JwtBuilder jwtBuilder = Jwts.builder()
                .signWith(SignatureAlgorithm.HS256, jwtToken) // 签发算法，秘钥为jwtToken
                .setClaims(claims) // body数据，要唯一，自行设置
                .setIssuedAt(new Date()) // 设置签发时间
                .setExpiration(new Date(System.currentTimeMillis() + time));// 一小时的有效时间
        String token = jwtBuilder.compact();
        return token;
    }

    public static Map<String, Object> checkToken(String token){
        try {
            Jwt parse = Jwts.parser().setSigningKey(jwtToken).parse(token);
            return (Map<String, Object>) parse.getBody();
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;

    }

```





