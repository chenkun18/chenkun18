---
title: 注解@CrossOrigin
date: 2023-11-08 14:40:20
category: 后端
tags:
  - SpringBoot
  - SpringBoot注解
  - 跨域
---
# 作用
- 原理：利用spring的拦截器实现往响应头里添加`Access-Control-Allow-Origin`等响应头信息
- 用在类、方法上
```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CrossOrigin {
    /** @deprecated */
    @Deprecated
    String[] DEFAULT_ORIGINS = new String[]{"*"};
    /** @deprecated */
    @Deprecated
    String[] DEFAULT_ALLOWED_HEADERS = new String[]{"*"};
    /** @deprecated */
    @Deprecated
    boolean DEFAULT_ALLOW_CREDENTIALS = false;
    /** @deprecated */
    @Deprecated
    long DEFAULT_MAX_AGE = 1800L;

    @AliasFor("origins")
    String[] value() default {};

    @AliasFor("value")
    String[] origins() default {};

    String[] originPatterns() default {};

    String[] allowedHeaders() default {};

    String[] exposedHeaders() default {};

    RequestMethod[] methods() default {};

    String allowCredentials() default "";

    long maxAge() default -1L;
}
```

# 概念
- 浏览器的同源策略限制：是一个重要的安全策略
- 同源（同一个域）：URL的协议、主机 (域名) 、端口都一致
- 跨域：URL的协议、主机 (域名) 、端口有一个不同
- 解决方案：
  - `JSONP`：通过script标签没有跨域限制的特性，进行资源的请求和获取（需要目标服务器进行配合，且仅支持get请求）
  - `CORS`：`Cross-Origin Resource sharing`（跨域资源共享），是一种基于HTTP头的机制，服务端在响应头里添加`Access-Control-Allow-Origin`等响应头信息标记哪些域可以访问
  - `服务器代理`：同源策略主要是限制浏览器和服务器之间的请求，服务器与服务器之间并不存在跨域


