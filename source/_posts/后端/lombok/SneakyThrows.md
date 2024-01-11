---
title: 注解@SneakyThrows
date: 2023-11-08 14:40:20
category: 后端
tags:
  - lombok注解
---

# 作用
- `@SneakyThrows`注解为代码生成一个`tr{}catch{}`块,并把异常向上抛出来
- 作用在方法、构造方法上
```java
package lombok;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
@Retention(RetentionPolicy.SOURCE)
public @interface SneakyThrows {
    Class<? extends Throwable>[] value() default {Throwable.class};
}
```
- java中常见的异常有两种：
    - `Exception`即非运行时异常(编译异常)
    - `RuntimeException`即运行时异常
- 正常情况：编译异常需要处理（增加`tr{}catch{}`或者方法抛出去）
```
public void hhh(User user){
  try {
    int userId = user.getId();
  } catch (Exception e) {
    throw new RuntimeException(e);
  }
}
```
- `@SneakyThrows`注解会自动为代码生成一个上面的`tr{}catch{}`块,并把异常向上抛出来（编辑器不会报错）
```
@SneakyThrows
public void hhh(User user){
   int userId = user.getId();
}
```
