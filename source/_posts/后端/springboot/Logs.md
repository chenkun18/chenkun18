---
title: 日志
date: 2023-11-10 13:48:51
category: 后端
tags:
  - SpringBoot
  - SpringBoot项目必备
  - SpringBoot日志
  - 系统日志
  - 操作日志
---
# 日志
## 系统日志
- 程序执行时输出的debug、info、warn、error等不同级别的程序执行记录信息；
- 给程序员或运维看的，一般在出现异常问题的时候，可以通过系统日志中记录的关键参数信息和异常提示，快速排除故障。
## 操作日志
- 用户实际业务操作行为的记录，这些信息一般存储在数据库里，如什么时间哪个用户点了某个菜单、修改了哪个配置等这类业务操作行为，这些日志信息是给普通用户或系统管理员看到。
- 需求分析
  - 1、记录用户的业务操作行为，记录的字段有：操作人、操作时间、操作功能、日志类型、操作内容描述、操作内容报文、操作前内容报文
  - 2、提供一个可视化的页面，可以查询用户的业务操作行为，对重要操作回溯；
  - 3、提供一定的管理功能，必要的时候可以对用户的误操作回滚；
- 反面实现
  - 1、每个接口里都加一段记录业务操作日志的记录；
  - 2、每个接口里都要捕获一下异常，记录异常业务操作日志；
  - 硬编码实现的业务操作日志管理功能的问题
    - 业务操作日志收集与业务逻辑耦合严重
    - 代码重复，新开发的接口在完成业务逻辑后要织入一段业务操作日志保存的逻辑
    - 已开发上线的接口，还要重新再修改织入业务操作日志保存的逻辑并测试，且每个接口需要织入的业务操作日志保存的逻辑是一样的
- 正面实现
  - 方案一：javax.servlet.Filter（过滤器）
    - 依赖于servlet容器
    - 基于函数回调实现
    - 拦截URL对应的请求request和响应response
    - 链式传递请求request和响应response
    - 结论：【适合处理请求内容和响应内容】【不适用细节记录】【获取不到特定方法，例如某个注解的标注方法】
  - 方案二：org.springframework.web.servlet.HandlerInterceptor（拦截器）
    - 依赖于SpringMVC框架
    - 基于Java的反射机制实现（AOP思想）
    - 拦截Controller中具体方法（请求处理程序）
    - 用来做自定义预处理(带有禁止执行处理程序本身的选项)和自定义后处理，例如公共处理程序代码和授权检查
    - 结论：【获取不到请求body的内容】【@ResponseBody标注的处理方法，获取不到响应数据】【可以获取特定方法，例如某个注解的标注方法】
    ```text
    - 其他概念
      - DispatcherServlet：【Spring的唯一Servlet】【使用HandlerMapping、HandlerAdapter进行请求分发】
      - HandlerMapping：【处理器映射器】【请求找到Handler处理程序】
      - HandlerAdapter：【处理器适配器】【执行Handler处理程序】
    - 为什么获取不到请求body？
      - Servlet创建HttpServletRequest时不会获取getInputStream()，没有将流传递给拦截器
      - Controller中能获取到是因为执行处理程序之前调用getInputStream()，将流传递过去了
    - 为什么这么设计？
      - 设计上就是如此【提高效率、降低耦合度、提高灵活性】
      - 解决：【1、自定义重写一个HttpServletRequestWrapper获取流参数】【2、自定义重写一个DispatcherServlet派发请求】
    ```
  - 方案三：Spring AOP（切面）
    - 对作用域没有限制，定义好切点
    - 颗粒度更细：前置通知（@Before）、后置通知（@After）、返回后通知（@AfterReturning）、异常通知（@AfterThrowing）、环绕通知（@Around）
    - 结论：【可以获取方法参数】【可以获取方法返回值】【可以修改返回值】

```java
package javax.servlet;

import java.io.IOException;

public interface Filter {

    /**
     * 过滤器实例初始化（只调用一次）
     * 
     * @param filterConfig 与正在初始化的过滤器实例相关联的配置信息
     *
     * @throws ServletException 初始化失败抛异常
     */
    default void init(FilterConfig filterConfig) throws ServletException {}

    /**
     * 每次请求时，容器都会调用doFilter方法
     * 此方法的典型实现将遵循以下模式:
     * 1。检查请求
     * 2。修改请求对象，请求头
     * 3。修改响应对象，响应头
     * 4-1。使用FilterChain对象chain. dofilter()调用链中的下一个实体，
     * 4-2。或者不将请求、响应对传递给过滤器链中的下一个实体以阻止请求处理
     *
     * @param request  要处理的请求
     * @param response 与请求相关联的响应
     * @param chain    链式调用，下一过滤器的访问
     *
     * @throws IOException      如果在此过滤器处理请求期间发生I/O错误
     * @throws ServletException 如果处理因任何其他原因失败
     */
    void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;

    /**
     * 由web容器调用，表示过滤器正在退出。
     * 此方法可以来清理所有被占用的资源(例如，内存、文件句柄、线程)，并确保任何持久状态都与过滤器在内存中的当前状态同步。
     */
    default void destroy() {}
}
```

```java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.lang.Nullable;
import org.springframework.web.method.HandlerMethod;

/**
 *
 * 有关更多选项和详细信息，请参见{@code org.springframework.web.servlet.AsyncHandlerInterceptor}
 * 通常，每个HandlerMapping bean定义一个拦截器链，共享其粒度。
 * 为了能够将某个拦截器链应用到一组处理程序，需要通过一个HandlerMapping bean映射所需的处理程序。拦截器本身在应用程序上下文中被定义为bean，由映射bean定义通过其“interceptors”属性引用。
 */
public interface HandlerInterceptor {

	/**
     * 在【HandlerAdapter调用处理程序】之前调用
     * 可以终止
     * 作用：登录验证（判断用户是否登录）权限验证：判断用户是否有权访问资源（校验token）
	 * @param request 当前HTTP请求
	 * @param response 当前HTTP响应
	 * @param handler 选择要执行的处理程序，用于类型或实例计算
	 * @return {@code true} 如果执行链应该继续下一个拦截器或处理程序本身。否则，DispatcherServlet假定这个拦截器已经处理了响应本身。
	 */
	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		return true;
	}

	/**
	 * 在【HandlerAdapter调用处理程序】之后调用
     * 在【DispatcherServlet呈现视图】之前调用
	 * 每个拦截器都可以对执行进行后处理，以执行链的相反顺序执行
     * 作用：将Controller层返回来的参数进行一些修改，在ModelAndView中
     * @param request 当前HTTP请求
     * @param response 当前HTTP响应
	 * @param handler 启动异步执行的处理程序(或{@link HandlerMethod})，用于类型或实例检查
	 * @param modelAndView 处理程序返回的{@code ModelAndView}(也可以是{@code null})
	 */
	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}

	/**
	 * 在【DispatcherServlet呈现视图】之后调用
     * 进行适当的资源清理。
     * 作用：例如登录的时候，我们经常把用户信息放到ThreadLocal中，为了防止内存泄漏，就需要将其remove掉，该操作就是在这里执行的
	 * 注意: 只有{@code preHandle}方法成功完成并返回{@code true}时才会被调用!
	 * 与{@code postHandle}方法一样，该方法将以相反的顺序在链中的每个拦截器上被调用，因此第一个拦截器将是最后一个被调用的。
	 *
     * @param request 当前HTTP请求
     * @param response 当前HTTP响应
	 * @param handler 启动异步执行的处理程序(或{@link HandlerMethod})，用于类型或实例检查
	 */
	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {}
}
```

## Spring AOP（切面）实现
- 依赖
```xml
<!-- aop依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
- 切入点：自定义注解
```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface JQHLog {
    /**
     * 功能名称
     */
    String name() default "";
    /**
     * 功能描述
     */
    String descript() default "";
}
```
- 切面
```java
@Component
@Aspect
@Slf4j
public class LogAspect implements Ordered {
  /**
   * 前后端不分离架构，如果统一异常处理执行顺序在@AfterThrowing之前，会出现@AfterThrowing中不执行情况。
   * 前后端分离架构不会出现此问题。
   * 解决：实现Ordered接口，并重写getOrder()方法，使其返回值为1，返回值越小，执行的顺序越靠前，使其执行顺序优先于全部异常处理类。
   */
  @Override
  public int getOrder() {
    return 1;
  }

  /**
   * 定义切点：自定义注解
   */
  @Pointcut("@annotation(com.example.uidemo.log.JQHLog)")
  public void pointcut(){}

  /**
   * 目标方法执行之前
   * @param joinPoint
   */
  @Before("pointcut()")
  public void before(JoinPoint joinPoint){
    print("@Before", joinPoint);
  }

  /**
   * 目标方法执行之后
   * @param joinPoint
   */
  @After("pointcut()")
  public void after(JoinPoint joinPoint){
    print("@After", joinPoint);
  }

  /**
   * 目标方法返回执行结果之后（可以修改返回值）
   * @param joinPoint
   */
  @AfterReturning(pointcut = "pointcut()", returning = "obj")
  public Object afterReturning(JoinPoint joinPoint, Object obj){
    print("@AfterReturning", joinPoint);
    return obj;
  }

  /**
   * 目标方法抛出异常后
   * @param joinPoint
   * @param e
   */
  @AfterThrowing(value = "pointcut()",throwing = "e")
  public void afterThrowing(JoinPoint joinPoint,Exception e){
    print("@AfterThrowing", joinPoint);
  }

  /**
   * 可以使用ProceedingJoinPoint joinPoint，获取目标对象，通过动态代理，代理目标类执行，在目标对象执行前后均可
   * @param joinPoint
   * @throws Throwable
   */
  @Around("pointcut()")
  public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    Object result = null;
    long beginTime = System.currentTimeMillis();
    try {
      result = joinPoint.proceed();
    } catch (Throwable e) {
      throw new RuntimeException("执行失败",e);
    }
    // 执行时长(毫秒)
    long time = System.currentTimeMillis() - beginTime;

    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    // 获取方法注解
    JQHLog logAnnotation = method.getAnnotation(JQHLog.class);
    // 注解名
    String name = logAnnotation.name();
    // 注解描述
    String descript = logAnnotation.descript();
    // 类名
    String className = joinPoint.getTarget().getClass().getName();
    // 方法名
    String methodName = joinPoint.getSignature().getName();
    // 请求的方法参数值
    Object[] args = joinPoint.getArgs();
    // 请求的方法参数名称
    LocalVariableTableParameterNameDiscoverer u = new LocalVariableTableParameterNameDiscoverer();
    String[] paramNames = u.getParameterNames(method);
    if (args != null && paramNames != null) {
      String params = "";
      for (int i = 0; i < args.length; i++) {
        params += "  " + paramNames[i] + ": " + args[i];
      }
    }
    // 请求request参数
    ServletRequestAttributes attributes = (ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
    // 请求ip
    String ip = attributes.getRequest().getRemoteHost();
    // 请求路径
    String requestURI = attributes.getRequest().getRequestURI();
    // 请求方法
    String requestMethod = attributes.getRequest().getMethod();
    print("@Around", joinPoint);
    return result;
  }

  /**
   * 打印基本信息
   * @param msg
   * @param joinPoint
   */
  public void print(String msg, JoinPoint joinPoint){
    Map info = new HashMap<>();
    info.put("类名", joinPoint.getTarget().getClass().getName());
    info.put("方法名", joinPoint.getSignature().getName());
    // 请求参数
    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    if (attributes != null){
      info.put("请求ip", attributes.getRequest().getRemoteHost());
      info.put("请求路径", attributes.getRequest().getRequestURI());
      info.put("请求方法", attributes.getRequest().getMethod());
    }
    log.info("{}：{}", msg, info.toString());
  }
}
```

- 可能遇到的问题
  - 单个类内的方法调用是不能够进入切面中的（@Transactional也有这问题）
  - 原因：内部方法调用时并未使用代理对象进行代理
  - 解决方法：内部调用时，拿到代理对象调用（用AopContext类来获取当前类的代理对象，前提：启动类上加`@EnableAspectJAutoProxy(exposeProxy = true)`）
```text
/**
 * 强制获取代理对象，必须开启exposeProxy配置，否则获取不到当前代理对象
 * @EnableAspectJAutoProxy(exposeProxy = true)
 */
private XXXImpl getThis() {
    return AopContext.currentProxy() != null ? (XXXImpl) AopContext.currentProxy() : this;
}
```



