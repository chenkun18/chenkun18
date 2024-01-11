---
title: EnableAspectJAutoProxy
date: 2024-01-02 12:47:32
category: 后端
tags:
  - SpringBoot
  - SpringBoot注解
  - AOP切面
---

# @EnableAspectJAutoProxy
- 切面配置注解`@EnableAspectJAutoProxy`
  - `proxyTargetClass`设置代理类型
  - `exposeProxy`设置代理类是否可以通过`AopContext`访问
```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	/**
	 * 是否要创建基于子类(CGLIB)的代理，而不是基于标准Java接口的代理。默认值是{@code false}。
	 */
	boolean proxyTargetClass() default false;

	/**
	 * 表明代理应该由AOP框架作为ThreadLocal公开，以便通过AopContext进行检索。
	 * 默认为关闭，即AopContext访问不到。
	 */
	boolean exposeProxy() default false;

}
```
