---
title: RestControllerAdvice
date: 2023-12-18 09:07:28
tags:
  - SpringBoot
  - SpringBoot项目必备
  - 全局异常处理
  - 统一响应
---
# @RestControllerAdvice
## 状态码enum类
```java
public enum ResultCode {

    SUCCESS(200,"操作成功!"),
    FAILURE(201,"操作失败"),
    /**系统相关的错误码：5开头**/
    ERROR(500,"系统异常，请稍后重试"),
    /**参数相关的错误码：1开头**/
    PARAM_ERROR(1000,"参数异常"),

    /**权限相关的错误码：2开头**/
    INVALID_TOKEN(2001,"访问令牌不合法"),
    ACCESS_DENIED(2002,"没有权限访问该资源"),
    USERNAME_OR_PASSWORD_ERROR(2003,"用户名或密码错误");

    private final int code;

    private final String message;

    ResultCode(int code, String message){
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```
## 统一返回值类
```java
@Data
public class Result<T> implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;

    private Integer code;

    private String message;

    private T data;

    public Result(Integer code,String message){
        this.code = code;
        this.message = message;
    }

    public Result(Integer code,String message,T data){
        this(code,message);
        this.data = data;
    }

    public static <T> Result<T> success(){
        return new Result<>(ResultCode.SUCCESS.getCode(),ResultCode.SUCCESS.getMessage());
    }

    public static <T> Result<T> success(T data){
        return new Result<>(ResultCode.SUCCESS.getCode(),ResultCode.SUCCESS.getMessage(),data);
    }

    public static <T> Result<T> fail(){
        return new Result<>(ResultCode.FAILURE.getCode(),ResultCode.FAILURE.getMessage());
    }

    public static <T> Result<T> fail(Integer code,String message){
        return new Result<>(code,message);
    }

    public static <T> Result<T> error(){
        return new Result<>(ResultCode.ERROR.getCode(),ResultCode.ERROR.getMessage());
    }
}
```
## 自定义运行时异常类
```java
public class CustomException extends RuntimeException{
    protected Integer code;
    protected String message;


    public CustomException(Integer code,String message,Throwable e) {
        super(message,e);
        this.code = code;
        this.message = message;
    }

    public CustomException(Integer code,String message){
        this(code,message,null);
    }

    public void setCode(Integer code){
        this.code = code;
    }
    public void setMessage(String message){
        this.message = message;
    }

    public Integer getCode(){
        return this.code;
    }

}
```
## 全局异常处理类
```java
@Slf4j
@RestControllerAdvice
public class GlobalException {
    /**
     * 自定义异常处理
     * @param e
     * @return
     */
    @ExceptionHandler(value = CustomException.class)
    public Result<Void> customExceptionHandler(CustomException e) {
        return Result.fail(e.getCode(),e.getMessage());
    }

    /**
     * 其他异常处理
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    public Result<Void> exceptionHandler(Exception e) {
        log.error("全局异常信息 ex={}", e.getMessage(), e);
        return Result.error();
    }
}
```
## 示例
```text
@GetMapping("/test")
public Result<String> test(){
    throw  new CustomException(ResultCode.USERNAME_OR_PASSWORD_ERROR.getCode(),ResultCode.USERNAME_OR_PASSWORD_ERROR.getMessage());
}
```