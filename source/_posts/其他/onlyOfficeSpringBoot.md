---
title: onlyOfficeSpringBoot
date: 2023-11-16 13:45:27
category: 后端
tags:
  - onlyOffice
  - onlyOfficeSpringBoot
---
# 类分析
- Thymeleaf：`https://blog.csdn.net/SoulNone/article/details/127572997`
- JPA：`https://blog.csdn.net/zdwzzu2006/article/details/131751921`
- H2：`https://blog.csdn.net/qq_29645505/article/details/97697093`
- JSON.simple：`https://blog.csdn.net/fireroll/article/details/48708241`
- Gson：`https://zhuanlan.zhihu.com/p/451745696`
- JWT：`https://www.cnblogs.com/hlkawa/p/13675792.html`
- JackSon：`https://blog.csdn.net/oschina_40730821/article/details/124025086`
- devtools：`https://juejin.cn/post/7027647026510692360`
- devtools：`https://blog.csdn.net/weixin_43701894/article/details/129921286`
## FileUtility、DefaultFileUtility
- 作用：文件工具类，用于文件名后缀判断文件类型，文件大小限制
- `String getFileExtension(String url);`从URL获取文件扩展名（文件后缀）
- `long getMaxFileSize();`获取最大文件大小
- `List<String> getFileExts();`获取所有支持的文件扩展名（所有后缀）
- `String getFileNameWithoutExtension(String url);`获取没有扩展名的文件名
- `List<String> getConvertExts();`获取可转换的文件扩展名列表（所有后缀）
## FileStorageMutator、FileStoragePathBuilder、LocalFileStorage
- 文件存储Mutator，文件存储路径生成器，本地文件存储
- `String updateFile(String fileName, byte[] bytes);`更新文件（字节数据更新到文件）
- `void createMeta(String fileName, String uid, String uname);`创建文件元信息
## DocumentManager、DefaultDocumentManager
- 
## ServiceConverter、DefaultServiceConverter

## FileController
- 文件上传接口：【post】`/upload`
  - 入参：`文件`、`用户id`
  - 作用：用于用户自己上传一个文件，然后在线编辑
- 文件转换接口：【post】`/converter`
  - 入参：`Converter`、`用户id`、`语言`
- 文件删除接口：【post】`/delete`
  - 入参：`Converter`
- 文件下载接口：【post】`/download`
  - 入参：`Converter`


