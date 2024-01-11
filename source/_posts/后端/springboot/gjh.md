---
title: 应用国际化
date: 2024-01-04 12:59:23
categories:
  - 后端
  - 前端
tags:
  - SpringBoot
  - SpringBoot项目必备
  - SpringBoot国际化
  - SpringBoot I18n
---
# 应用国际化
## 前端国际化
- 页面显示和用户界面的本地化
- 方案：`React Intl`、`Vue I18n`、`Angular i18n`
## 后端国际化
- 处理与业务逻辑和数据相关的国际化
### 概念
- `java.util.Locale`类：本地化语言类，包含各个国家地区的语言languages
- `org.springframework.context.MessageSource`：其主要是根据`Locale`信息获取对应的国际化消息的集合，然后根据`code`获取对应的消息
- `org.springframework.web.servlet.LocaleResolver`：设置当前会话默认的国际化语言
### 步骤
- 1、创建国际化文件`resources/i18n/messages`
  - `message.properties`
  - `message_cn_ZH.properties`
  - `message_en_GB.properties`
- 2、yml指定国际化
```yaml
spring:
  messages:
    basename: i18n/messages
    encoding: UTF-8
```
- 3、创建国际化表`i18n_message`（除了可以存在配置文件，也可以存在数据库）

| 属性名 | 国家代码 | 对应内容 |
| ----- | ------ | ------- |
| `PASSWORD` | `cn_ZH` | `密码` |
| `PASSWORD` | `en_GB` | `password` |
| `AGE` | `cn_ZH` | `年龄` |
| `AGE` | `en_GB` | `age` |

- 4、表对应实体类以及Dao
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@TableName("i18n_message")
@ApiModel(description = "国际化表")
public class I18nMessage {

    @ApiModelProperty("属性名")
    private String code;

    @ApiModelProperty("国家代码")
    private String locale;

    @ApiModelProperty("对应内容")
    private String message;

}
```

```java
@Mapper
public interface I18nMessageMapper extends BaseMapper<I18nMessage> { }
```

- 5、自定义`MessageSource`类
```java
@Component("messageSource")
public class CustomMessageSource extends AbstractMessageSource implements InitializingBean {

    @Resource
    private I18nMessageMapper i18nMessageMapper;

    /**
     * 这个是用来缓存数据库中获取到的配置的（可以改到redis）
     * 数据库配置更改的时候可以调用reload方法重新加载
     */
    private static final Map<String, Map<String, String>> LOCAL_CACHE = new ConcurrentHashMap<>();

    /**
     * Bean属性初始化完成之后执行
     */
    @Override
    public void afterPropertiesSet() {
        this.reload();
    }

    /**
     * 重新加载消息到该类的Map缓存中
     */
    public void reload() {
        LOCAL_CACHE.clear();// 清除该类的缓存
        LOCAL_CACHE.putAll(this.loadAllMessageResources());// 加载所有的国际化资源
    }

    /**
     * 加载所有的国际化消息资源
     * 同时从数据库和properties文件中读取国际化信息
     * Map<LanguageCode, Map<code, 翻译>>
     */
    private Map<String, Map<String, String>> loadAllMessageResources() {
        // todo 从数据库中查询所有的国际化资源
        List<I18nMessage> allLocaleMessage = i18nMessageMapper.selectAllList();
        allLocaleMessage = Optional.OfNullable(allLocaleMessage).orElseGet(() -> new ArrayList<>());
        // 将查询到的国际化资源转换为 Map<地区码, Map<code, 信息>> 的数据格式
        Map<String, Map<String, String>> localeMsgMap = allLocaleMessage.stream().collect(Collectors.groupingBy(I18message::getLocale, Collectors.toMap(I18nMessage::getCode, I18nMessage::getMessage)));

        // 获取国家地区List
        List<Locale> localeList = localeMsgMap.keySet().stream().map(Locale::new).collect(Collectors.toList());
        for (Locale locale : localeList) {
            // todo 按照国家地区来读取本地的国际化资源文件,我们的国际化资源文件放在i18n文件夹之下
            ResourceBundle resourceBundle = ResourceBundle.getBundle("i18n/messages", locale);
            Set<String> keySet = resourceBundle.keySet();// 获取国际化资源文件中的key
            Map<String, String> msgFromFileMap = keySet.stream().collect(Collectors.toMap(Function.identity(), resourceBundle::getString));// 将 code=信息 格式的数据收集为 Map<code,信息> 的格式
            // 将本地的国际化信息和数据库中的国际化信息合并
            Map<String, String> localeFileMsgMap = localeMsgMap.get(locale.getLanguage());
            localeFileMsgMap.putAll(msgFromFileMap);// 配置文件会覆盖数据库的
            localeMsgMap.put(locale.getLanguage(), localeFileMsgMap);
        }

        return localeMsgMap;
    }

    /**
     * 缓存Map中get国际化资源
     */
    private String getSourceFromCacheMap(String code, Locale locale) {
        String language = ObjectUtils.isEmpty(locale) ? LocaleContextHolder.getLocale().getLanguage() : locale.getLanguage();
        // 获取缓存中对应语言的所有数据项
        Map<String, String> propMap = LOCAL_CACHE.get(language);
        // 找到直接返回，找不到返回code
        return Optional.OfNullable(propMap.get(code)).orElse(code);
    }

    /**
     * 实现方法，供AbstractMessageSource.getMessage方法内部使用
     */
    @Override
    protected MessageFormat resolveCode(String code, Locale locale) {
        String msg = this.getSourceFromCacheMap(code, locale);
        return new MessageFormat(msg, locale);
    }

    /**
     * 实现方法，供AbstractMessageSource.getMessage方法内部使用
     */
    @Override
    protected String resolveCodeWithoutArguments(String code, Locale locale) {
        return this.getSourceFromCacheMap(code, locale);
    }

}
```

- 6、自定义`LocaleResolver`国际化区域解析器，解析url或者header中的参数
```java
@Component
public class CustomLocaleResolver implements LocaleResolver {
   /**
    * 根据当前请求解析当前请求的本地化信息
    */
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        String l = httpServletRequest.getParameter("lang");// 请求url参数
        String header = httpServletRequest.getHeader("lang");// 请求头参数
        Locale locale = null;// 国际化，每一个locale对象都代表一个特定的政治文化，地区和创建方法        
        return Optional.OfNullable(l).map(i -> {
          // 判断请求url参数中是否有带lang参数（优先）
          String[] split = l.split("_");// 根据下划线分割
          return new Locale(split[0], split[1]);// 创建国际化对象
        }).orElseGet(() -> {
          return Optional.OfNullable(header).map(i -> {
            // 判断请求头参数中是否有带lang参数
            header = header.replaceAll("\"","");// 替换为空
            String[] split = header.split("_");// 根据下划线分割
            return new Locale(split[0], split[1]);// 创建国际化对象
          }).orElse(null);
        });
    }

   /**
    * 设置当前请求、响应的本地化信息
    */
    @Override
    public void setLocale(HttpServletRequest httpServletRequest, @Nullable HttpServletResponse httpServletResponse, @Nullable Locale locale) {
    }

}
```

- 7、封装工具类
```java
@Component
public class I18nMessageUtil {

    private final MessageSource messageSource;

    /**
     * @param code 对应messages配置的key
     * @return String
     */
    public String getMessage(String code){
        return getMessage(code, null);
    }

    /**
     * @param code 对应messages配置的key
     * @param args 数组参数
     * @return String
     */
    public String getMessage(String code,Object[] args){
        return getMessage(code, args, "");
    }


    /**
     * @param code 对应messages配置的key
     * @param args 数组参数
     * @param defaultMessage 没有设置key的时候的默认值
     * @return String
     */
    public String getMessage(String code,Object[] args,String defaultMessage){
        return messageSource.getMessage(code, args, defaultMessage, LocaleContextHolder.getLocale());
    }

}
```

### 使用场景
- 1、全局响应状态码（code）对应的消息（msg）国际化
  - 法一：工具类中，将`MessageSource`做静态变量注入，提供几个静态方法进行国际化，在`new Result()`的时候调用静态方法进行国际化 【`@Component`+`有参构造函数@Autowired注入`】
  ```java
  @Component
  public class I18nMessageUtil {
  
      private static MessageSource messageSource;
  
      @Autowired
      public I18nMessageUtil(MessageSource messageSource){
          I18nMessageUtil.messageSource = messageSource;
      }
      /**
       * @param code 对应messages配置的key
       * @return String
       */
      public static String getMessage(String code){
          return getMessage(code, null);
      }
  
      /**
       * @param code 对应messages配置的key
       * @param args 数组参数
       * @return String
       */
      public static String getMessage(String code,Object[] args){
          return getMessage(code, args, "");
      }
  
  
      /**
       * @param code 对应messages配置的key
       * @param args 数组参数
       * @param defaultMessage 没有设置key的时候的默认值
       * @return String
       */
      public static String getMessage(String code,Object[] args,String defaultMessage){
          return messageSource.getMessage(code, args, defaultMessage, LocaleContextHolder.getLocale());
      }
  }
  ```
  - 法二：AOP切面，`@AfterReturning`中获取返回对象，修改里面的属性值（对象是不能修改的成一个新的滴）
- 2、接口参数校验结果国际化
  - `@NotBlank(message="{xxx_err_code}")`
