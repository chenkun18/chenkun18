---
title: OnlyOffice介绍及docker部署Document Server
date: 2023-11-02 13:27:26
category: 其他
tags:
  - OnlyOffice
  - OnlyOffice Docker部署Document Server
---

# 描述和功能
## 描述
- OnlyOffice是一个多端协同的 Office 办公套件，结合了文本文档、电子表格、演示文稿和表单的编辑器。
- 后端：JavaScript语言和Node.js构建服务器端脚本
- 前端：使用HTML5呈现文档元素。
  - 编辑器的技术基础是HTML5 Canvas。画布是一个包含不同绘图元素的容器，如线条、形状、图形和框架。HTML结构中的每个对象都是独立处理的，而画布结构的其余部分不受该特定对象的操作的影响。这有助于保持高精度的显示，使内容浏览器和操作系统不可知，并在查看、编辑或打印文档时提供平等的文档呈现。
- 核心格式组是OOXML，它使套件与DOCX、XLSX和PPTX文件完全兼容，而其他格式则通过内部转换进行处理。
- 具有`实时协同编辑`、`评论`、`在线聊天`、`版本历史记录`、`跟踪更改`和`文档比较`等协作工具，并支持各种类型的共享权限，包括`完全访问`、`仅查看`、`评论`、`审阅`、`填写表单`和`自定义筛选`。
- 可以使用基于API通信或WOPI协议的集成应用程序在第三方环境中集成。
- 可以使用不同的方法在客户端的体系结构上进行部署，包括Windows和Linux服务器安装选项、Docker、Pod
## 结构
- 服务体系结构在OnlyOffice文档部署模型的`数据安全性`、`性能`和`灵活性`方面发挥着至关重要的作用。
- 客户端：浏览器中调用一个完整的文档编辑器应用程序，将大部分流程负载转移到客户端
- 服务器端只处理`保存`、`传输文件更改`和`拼写检查`等过程。
- 体系结构中的每个服务都可以单独托管在一个集群中，这还增加了容错能力，并允许根据环境构建自定义服务映射。
- 服务架构如下：
![001architecture.png](001architecture.png)
- 这样的架构设计使得一台`32GB RAM的8核`机器足以容纳`1000个同时编辑的文档`，假设每4秒对所有文档进行一次更改（1000个连接每秒250次更改）。
## 编辑功能
### 支持的电子文件格式
支持以下类型的格式用于查看和编辑：

|  | 查看 | 编辑 | 下载 |
| :------ | :------ | :------ | :------ |
| 文本文档 | DOC, DOCX, DOTX, FB2, ODT, OTT, RTF, TXT, PDF, PDF/A, HTML, EPUB, XPS, DjVu, XML, DOCXF, OFORM | DOC, DOCX, DOTX, FB2, ODT, OTT, RTF, TXT, HTML, EPUB, XML, DOCXF, OFORM | DOCX, DOTX, FB2, ODT, OTT, RTF, TXT, PDF, PDF/A, HTML, EPUB, DOCXF, OFORM |
| 电子表格 | XLS, XLSX, XLTX, ODS,OTS, CSV | XLS, XLSX, XLTX, ODS,OTS, CSV|XLSX, XLTX, ODS,OTS, CSV, PDF,PDF/A |
| 演讲 | PPT, PPTX, POTX, ODP, OTP, PPSX | PPT, PPTX, POTX, ODP, OTP|PPTX, POTX, ODP, OTP, PDF, PDF/A, PNG, JPG |

## 协作功能
- 允许在集成平台内的任意数量的用户之间协作编辑文档。
- 由于OnlyOffice Docs的架构意味着编辑器的客户端性能，在同一服务器上处理同一文档时，每个用户都可以独立于所有其他用户激活任何功能或选择任何模式。
- 这显著改进了许多方面的工作，包括使用`撤消`和`重做`命令独立恢复操作。这种方法还最小化了服务器负载，使设置变得轻量级。
- 协作功能包括：
  - 灵活的共享权限：仅查看、完全访问、审阅、注释、填写表单、自定义筛选器。
  - 两种协同编辑模式：快速（实时）和严格（段落锁定）
  - 跟踪变化；
  - 评论；
  - 在评论中提及用户；
  - 内置聊天；
  - 版本和修订控制（版本历史）。
- 通过独立使用所有功能和协作模式，来自不同设备和客户端（包括网络套件、桌面和移动应用程序）的大量用户之间可以进行协作。
- 尽管大多数操作都是在OnlyOffice文档（文档服务器）中执行的，但有些功能在集成中依赖于与用户管理系统的通信。版本历史记录和提及等功能的可用性目前在每个集成中都有所不同。

## 数据安全
- 在设计上是安全的，这得益于自托管的部署模型。此外，在任何基础结构中存储和编辑文件时，它都提供了多种保护数据的功能：
  - 为了保护流量，OnlyOffice使用HTTPS运行，而文档编辑连接则使用JWT进行额外保护。
  - 共享中提供灵活的权限：使用完全访问、只读、评论、审阅或填写表单权限，允许或不允许修改电子表格中的筛选器，如果您愿意，还可以限制下载、复制和打印。
  - 您还可以应用水印，以避免未经授权重新分发您的内容。
  - 文件加密可用于文件和电子表格，包括保护整个工作簿和电子表格中的单独表格。
- OnlyOffice具有内置功能，可帮助企业遵守HIPAA和欧盟隐私法规。

## 接口定制
可以自定义OnlyOffice文档的界面，以增强使用套件时的体验。

| 选项 | 说明 |
| :------ | :------ |
| 界面主题 | 可以将界面主题分别设置为`暗`、`亮`和`经典`灯光。在某些集成中，可以在管理级别控制主题设置。 |
| 界面和文档缩放 | 编辑器支持100%、125%、150%、175%和200%的自动缩放。 手动文档缩放选项的范围从50%到200%。 |
| 工具栏、状态栏和标尺 | 可以隐藏主工具栏、状态栏和标尺以展开工作区域。 |
| 工具栏布局* | 在设置中，可以在完整和紧凑的工具栏布局之间切换，并更改选项卡的颜色。 |
| 附加选项（适用于开发人员） | 脚本级配置允许调整按钮和命令、公司和联系人信息、徽标等的显示。 |

# 整合机制
## 通过基于API的连接器进行集成
- 基于API的集成是将OnlyOffice文档连接到文件存储和共享环境的一种随意方式，包括同步和共享服务、内容管理系统（CMS）和学习管理系统（LMS）。目前大多数可用的连接器都是基于API的。
- 开放式API允许构建应用程序，以集成OnlyOffice独有的任何功能，或将自定义重新构建功能集成到连接的存储中，为用户提供完整的体验。
- 基于API的集成方案
![002integration.png](002integration.png)
- OnlyOffice文档（文档服务器）与第三方服务的集成需要一个额外的应用程序，该应用程序将数据转换为兼容的格式。此角色由连接器扮演。OnlyOffice团队创建自己的官方连接器，并协助合作伙伴和第三方开发人员创建此类应用程序。
- 集成应用程序结构
![003structure.png](003structure.png)
- 通过基于API的集成提供的功能*：

| 功能      | 说明                                                                                                                       |
|:--------|:-------------------------------------------------------------------------------------------------------------------------|
| 支持格式    | 用于查看和编辑：DOCX、XLSX、PPTX、PPSX、OFORM、DOCXF 仅供查看：PDF、DJVU、TXT、CSV、ODT、ODS、ODP、DOC、XLS、PPT、PPS、EPUB、RTF、HTML、HTM、MHT、XPS      |
| 协作模式    | 在实时和段落锁定协同编辑模式之间切换。                                                                                                      |
| 自定义     | 为编辑器设置界面语言和主题、隐藏聊天菜单按钮、更改“关于”部分中的信息、界面自定义，如调整标题和工具栏、品牌、连接插件                                                              |
| 基本功能    | 查看、编辑、共同编辑、移动查看和编辑、简化查看（嵌入式）                                                                                             |
| 其他操作：方法 | 以所选格式下载文件、将文件标记为收藏夹、显示带有消息的工具提示                                                                                          |
| 其他操作：事件 | 关闭编辑器、打开文件位置、将文件从查看模式切换到编辑模式、重命名文档、管理文档访问权限、打开版本历史记录、插入存储中的图像、邮件合并、与存储中的文档进行比较、在书签位置获取打开文件的链接、以所需格式保存文件、在评论中提及其他用户、创建新文档 |
| 安全      | IP地址白名单、使用JWT验证请求和防止未经授权访问                                                                                               |
| 文档权限    | 查看、编辑、审查（文本文件）、评论、填写表格、修改内容控件（文本文档）、修改过滤器（电子表格）、复制到剪贴板、下载、打印、重命名                                                         |
| 局限性     | 编辑器中提供的所有功能都可以集成。                                                                                                        |

# 实战开发
## OnlyOffice社区版
- 文档服务器：`Document Server`
- 社区服务器：`Community Server`
- 邮件服务器：`Mail Server`
## docker部署文档服务器
- 【[官网教程](https://helpcenter.onlyoffice.com/installation/docs-community-install-docker.aspx)】
- 1、拉取镜像：
```
docker pull onlyoffice/documentserver
```
- 2、启动容器：
```shell
docker run -itd -p 9000:80 --restart=always 
-v D:\workNew\study\onlyoffice\DocumentServer\logs:/var/log/onlyoffice # 日志数据
-v D:\workNew\study\onlyoffice\DocumentServer\data:/var/www/onlyoffice/Data # ssl证书
-v D:\workNew\study\onlyoffice\DocumentServer\lib:/var/lib/onlyoffice # 文件缓存数据
-v D:\workNew\study\onlyoffice\DocumentServer\db:/var/lib/postgresql # 数据库
-e JWT_SECRET=my_jwt_secret #  JWT密钥，不设置会自动生成，也可以关闭【-e JWT_ENABLED=false】
onlyoffice/documentserver # 镜像名
```

- 进入容器：`docker exec -it ID /bin/bash`
- 启动所有的内置服务：`supervisorctl restart all`
- 退出容器：`exit`
- 访问：`http://IP:9000/`
![004page.png](004page.png)
- 页面包括
  - [开发文档](https://api.onlyoffice.com/editors/basic)
  - 查看密钥：
  ```
  docker exec ID /var/www/onlyoffice/documentserver/npm/json -f /etc/onlyoffice/documentserver/local.json 'services.CoAuthoring.secret.session.string'
  ```
  - 启动example服务：
  ```
  docker exec ID sudo supervisorctl start ds:example
  ```
  - 设置example服务自动启动：
  ```
  docker exec ID sudo sed 's,autostart=false,autostart=true,' -i /etc/supervisor/conf.d/ds-example.conf
  ```
- 服务列表（都是通过pkg打包成可执行文件，摆脱node环境的依赖）
  - `ds:converter`：转换器
  - `ds:docservice`：文档服务
  - `ds:example`：例子
  - `ds:metrics`
- 服务占用端口：
  - `nginx`：`80`
  - `rabbitmq`：`5672`、`25672`、`4369`
  - `postgres`：`5432`
  - `docservice`: `8000`
  - `example`：`3000`
![005port.png](005port.png)

## example测试
![006example.png](006example.png)
![007example-page.png](007example-page.png)


