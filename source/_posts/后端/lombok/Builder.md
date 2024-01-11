---
title: 注解@Builder
date: 2023-11-09 10:52:09
category: 后端
tags:
  - lombok注解
---

# 作用
- 它作用于类，将其变成建造者模式，可以以链的形式调用
```java
package lombok;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD, ElementType.CONSTRUCTOR})
@Retention(RetentionPolicy.SOURCE)
public @interface Builder {
    String builderMethodName() default "builder";

    String buildMethodName() default "build";

    String builderClassName() default "";

    boolean toBuilder() default false;

    AccessLevel access() default AccessLevel.PUBLIC;

    String setterPrefix() default "";

    @Target({ElementType.FIELD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Default {
    }

    @Target({ElementType.FIELD, ElementType.PARAMETER})
    @Retention(RetentionPolicy.SOURCE)
    public @interface ObtainVia {
        String field() default "";

        String method() default "";

        boolean isStatic() default false;
    }
}
```
- 原理：
  - 编译时创建一个内部静态类，有和实体类相同的属性（构造器）
  - 构造器：所有属性的set方法，返回值是this（链式调用）


