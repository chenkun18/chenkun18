---
title: Spring Bean注入
date: 2023-12-01 12:44:49
category: 后端
tags:
  - SpringBoot
  - Bean注入
---
# spring bean注入
- `@Resource`：【`javax.annotation.Resource`】【注入可变的依赖】【作用在字段、方法和构造函数】【可根据名称显示匹配注入】【没找到抛异常】
- `@Autowired`：【`org.springframework.beans.factory.annotation.Autowired`】【注入可变的依赖】【作用在字段、方法和构造函数】【没找到为null】
- `private final`：【Spring Boot中使用构造函数注入】【注入不可变的依赖，确保字段的安全性和线程安全性】
