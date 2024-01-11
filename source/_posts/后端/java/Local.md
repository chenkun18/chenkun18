---
title: Local类
date: 2023-11-09 15:03:28
category: 后端
tags:
  - java.util类
  - java国际化
---

# 作用
- 不同国家的语言、文字、数字、日期都有不同显示
- Local可用根据系统语言对其进行显示
```java
import java.text.NumberFormat;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;
import java.util.Locale;
import java.util.Set;

public class Test {
    public static void main(String[] args) {
        /**
         * 获取当前的语言和国家
         */
        Locale defaultLocal = Locale.getDefault();
        String country = defaultLocal.getCountry();// CN
        String displayCountry = defaultLocal.getDisplayCountry();// 中国
        String displayCountry1 = defaultLocal.getDisplayCountry(Locale.ENGLISH);// China
        String iso3Country = defaultLocal.getISO3Country();// CHN
        String language = defaultLocal.getLanguage();// zh
        String iso3Language = defaultLocal.getISO3Language();// zho
        String displayLanguage = defaultLocal.getDisplayLanguage();// 中文
        String displayLanguage1 = defaultLocal.getDisplayLanguage(Locale.ENGLISH);// Chinese
        String displayName = defaultLocal.getDisplayName();// 中文 (中国)
        /**
         * 获取所有的语言及国家
         */
        Locale[] locales = Locale.getAvailableLocales();
        /**
         * 获取所有国家
         */
        String[] countries = Locale.getISOCountries();
        /**
         * 获取所有的语言
         */
        String[] languages = Locale.getISOLanguages();
        /**
         * 获取某个国家对应的标签
         */
        String s = Locale.US.toLanguageTag();// en-US
        /**
         * 获取某个国家对应的语言
         */
        String s2 = Locale.ENGLISH.toLanguageTag();// en
        /**
         * 构建Local
         */
        Locale locale = Locale.forLanguageTag("en-US");// 使用标签生成Local
        Locale locale2 = new Locale("en");// 构造函数生成Local
        Locale locale3 = new Locale("en", "US");// 构造函数生成Local
        Locale locale4 = Locale.CHINA;
        /**
         * 数字格式化
         */
        NumberFormat numberInstance = NumberFormat.getNumberInstance(Locale.CHINA);
        String format = numberInstance.format(123456.78);// 123,456.78
        /**
         * 货币格式化
         */
        NumberFormat currencyInstance = NumberFormat.getCurrencyInstance(Locale.CHINA);
        String format1 = currencyInstance.format(123456.78);// ￥123,456.78
        /**
         * 百分比格式化
         */
        NumberFormat percentInstance = NumberFormat.getPercentInstance(Locale.CHINA);
        String format2 = percentInstance.format(0.2);//
        /**
         * 时间日期格式化
         * - 月份和星期的数字应该用本地语言
         * - 年月日的顺序符合本地习惯
         * - 公历可能并不是本地首选表示方法
         * - 需考虑本地的时区
         */
        LocalDate localDate = LocalDate.now();// 2023-11-09
        String format3 = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT).format(localDate);// 23-11-9
        String format31 = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT).withLocale(Locale.ENGLISH).format(localDate);// 11/9/23
        String format4 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM).format(localDate);// 2023-11-9
        String format41 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM).withLocale(Locale.ENGLISH).format(localDate);// Nov 9, 2023
        String format5 = DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG).format(localDate);// 2023年11月9日
        String format51 = DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG).withLocale(Locale.ENGLISH).format(localDate);// November 9, 2023
        String format6 = DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).format(localDate);// 2023年11月9日 星期四
        String format61 = DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).withLocale(Locale.ENGLISH).format(localDate);// Thursday, November 9, 2023

        LocalTime localTime = LocalTime.now();// 16:31:18.072
        String format7 = DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT).format(localTime);// 下午4:29
        String format71 = DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT).withLocale(Locale.ENGLISH).format(localTime);// 4:29 PM
        String format8 = DateTimeFormatter.ofLocalizedTime(FormatStyle.MEDIUM).format(localTime);// 16:29:49
        String format81 = DateTimeFormatter.ofLocalizedTime(FormatStyle.MEDIUM).withLocale(Locale.ENGLISH).format(localTime);// 4:29:49 PM
        String format9 = DateTimeFormatter.ofLocalizedTime(FormatStyle.LONG).format(localTime);// 下午04时29分49秒
//        String format91 = DateTimeFormatter.ofLocalizedTime(FormatStyle.LONG).withLocale(Locale.ENGLISH).format(localTime);//
//        String format10 = DateTimeFormatter.ofLocalizedTime(FormatStyle.FULL).format(localTime);//
//        String format101 = DateTimeFormatter.ofLocalizedTime(FormatStyle.FULL).withLocale(Locale.ENGLISH).format(localTime);//

        LocalDateTime localDateTime = LocalDateTime.now();// 2023-11-09T16:31:18.072
        String format11 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT).format(localDateTime);// 23-11-9 下午4:31
        String format111 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT).withLocale(Locale.ENGLISH).format(localDateTime);// 11/9/23 4:31 PM
        String format12 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).format(localDateTime);// 2023-11-9 16:31:18
        String format121 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).withLocale(Locale.ENGLISH).format(localDateTime);// Nov 9, 2023 4:31:18 PM
        String format13 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG).format(localDateTime);// 2023年11月9日 下午04时31分18秒
//        String format131 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG).withLocale(Locale.ENGLISH).format(localDateTime);//
//        String format14 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).format(localDateTime);//
//        String format141 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).withLocale(Locale.ENGLISH).format(localDateTime);//

        ZonedDateTime zonedDateTime = ZonedDateTime.now();// 2023-11-09T16:31:54.917+08:00[GMT+08:00]
        String format15 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT).format(zonedDateTime);// 23-11-9 下午4:31
        String format151 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT).withLocale(Locale.ENGLISH).format(zonedDateTime);// 11/9/23 4:31 PM
        String format16 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).format(zonedDateTime);// 2023-11-9 16:31:54
        String format161 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).withLocale(Locale.ENGLISH).format(zonedDateTime);// Nov 9, 2023 4:31:54 PM
        String format17 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG).format(zonedDateTime);// 2023年11月9日 下午04时31分54秒
        String format171 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG).withLocale(Locale.ENGLISH).format(zonedDateTime);// November 9, 2023 4:31:54 PM GMT+08:00
        String format18 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).format(zonedDateTime);// 2023年11月9日 星期四 下午04时31分54秒 GMT+08:00
        String format181 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).withLocale(Locale.ENGLISH).format(zonedDateTime);// Thursday, November 9, 2023 4:31:54 PM GMT+08:00

    }
}
```


