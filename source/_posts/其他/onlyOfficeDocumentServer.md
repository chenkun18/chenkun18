---
title: OnlyOffice Document Server教程
date: 2023-11-03 15:58:33
category: 其他
tags:
  - OnlyOffice
  - OnlyOffice Document Server
---
# OnlyOffice Document Server教程
- [官网教程地址](https://api.onlyoffice.com/)
## 开始
### 基本概念
- OnlyOffice是一个开源办公套件，包括文本文档、电子表格、演示文稿和可填写表格的编辑器。它提供了以下功能：
  - 创建、编辑和查看文本文档、电子表格、演示文稿和可填写表格；
  - 多人实时协作处理文件。
- OnlyOffice还支持WOPI协议，该协议用于将您的应用程序与在线office集成。
- OnlyOffice API用于让开发人员将`OnlyOffice文档/电子表格/演示文稿编辑器`集成到自己的网站中，并设置和管理编辑器。
- API JavaScript文件通常可以在以下编辑器文件夹中找到：
  - `https://documentserver/web-apps/apps/api/documents/api.js`
  - 其中`documentserver`是Document server的服务器的地址。
- 要嵌入编辑器的目标HTML文件需要有一个占位符div标记，其中将传递有关编辑器参数的所有信息：
```html
<div id="placeholder"></div>
<script type="text/javascript" src="https://documentserver/web-apps/apps/api/documents/api.js"></script>
```
- 包含可变参数的页面代码如下所示：
```js
var docEditor = new DocsAPI.DocEditor("placeholder", config);
```
- 其中config是一个对象：
```js
config = {
    "document": {
        "fileType": "docx",
        "key": "Khirz6zTPdfd7",
        "title": "Example Document Title.docx",
        "url": "https://example.com/url-to-example-document.docx"
    },
    "documentType": "word",
    "editorConfig": {
        "callbackUrl": "https://example.com/url-to-callback.ashx"
    }
};
```
  - 其中`example.com`是文档管理和存储服务器地址。请参阅[如何工作](#开始)部分，以了解有关DocumentServer服务客户端-服务器交互的更多信息。
- 一切ok则`docEditor`对象可以用来调用文档编辑器方法。 
- 上面的示例包括正确启动所需的所有参数。
- 还可以更改其他非强制性参数，以实现其他功能（更改文档的访问权限、显示文档的不同信息等）。请参阅[高级参数](#开始)部分，了解这些参数是什么以及如何更改它们。
- 为了防止替换重要参数，请以令牌的形式向请求添加加密签名。
### 获取OnlyOffice文档
- 可用于Windows、Linux和Docker。
- 要在本地服务器上安装，请按照OnlyOffice帮助中心中的说明进行操作：
  - [Windows安装](https://helpcenter.onlyoffice.com/installation/docs-developer-install-windows.aspx)
  - [Linux安装](https://helpcenter.onlyoffice.com/installation/docs-developer-install-ubuntu.aspx)
  - [Docker安装](https://helpcenter.onlyoffice.com/installation/docs-developer-install-docker.aspx)
- 在使用API文档之前，如有必要，建议进行以下设置：
  - 配置文件中配置服务器设置；
  - 切换到HTTPS协议，以便使用SSL证书进行安全连接；
  - 添加额外的字体，以增强编辑器的工作；
  - 为应用程序界面添加您自己的颜色主题。
### 立即尝试
- 您可以打开各种文件类型进行编辑、查看、共同编辑、审查或查看重新命名的工作原理。
- 点击`</>`按钮查看相应的[示例源代码](https://api.onlyoffice.com/editors/try)。
### 特定于语言的例子
- [各个语言的demo](https://api.onlyoffice.com/editors/demopreview)

