---
title: hexo博客搭建
date: 2023-11-01
category: 其他
tags:
  - 博客搭建
---
# 什么是 Hexo？
- 一个快速、简洁且高效的博客框架【[Hexo中文官网](https://hexo.io/zh-cn/)】

# 本地环境
- nodejs
- git

# 本地项目搭建
- 1、安装hexo客户端：`npm install -g hexo-cli`
- 2、新建一个文件夹：`myblog`
- 3、初始化hexo，在文件夹下执行：`hexo init`
  - 目录结构
    - node_modules: 依赖包
    - scaffolds：生成文章的一些模板
    - source：用来存放你的文章
    - themes：主题
    - _config.yml: 博客的配置文件
    ![文章目录](001menu.png)
- 4、启动运行：`hexo server`
![启动运行](002start.png)
- 5、访问`http://localhost:4000/`
![访问界面](003visit.png)

# 新增主题
- 官网选择需要的主题【[主题选择网址](https://hexo.io/themes/)】
- 我这里选择`snippet`
![004theme.png](004theme.png)
- 进入`themes`目录下把代码拉下来：`git clone https://github.com/shenliyang/hexo-theme-snippet.git`
![005snippet.png](005snippet.png)
![006snippet.png](006snippet.png)
- `_config.yml`文件中配置主题：`theme: hexo-theme-snippet`
- 重新启动运行：`hexo server`
![007start.png](007start.png)
- 每种主题有哪些配置可以看md文件，根据自己需求进行配置
![008snippet.png](008snippet.png)

# 写博
- 新建分类：`hexo new page "categories"`
- 新建标签：`hexo new page "tags"`
- 新建文章：`hexo new 文件名`（会在_posts下生成md文件）
![009newmd.png](009newmd.png)
  - 文件内容（前面是配置标题、时间、分类、标签，后面就可以写文章了）
```md
---
title: 文件名
date: 2023-11-02 10:41:01
category: 哈哈
tags:
  - xxx
  - hhh
---
# 标题1
- 哈哈哈
```
![010newmd.png](010newmd.png)
- md文件中如何插入图片等资源【[官网资源配置](https://hexo.io/zh-cn/docs/asset-folders)】
  - 我是建一个和md文件名相同的文件夹，然后将图片放下面
  ![011config.png](011config.png)

# 部署到github
- 教程：【[官网教程](https://hexo.io/zh-cn/docs/github-pages)】
- 1、新建一个github仓库，仓库名为`账号名.github.io`，新建md文件初始化`main`分支
- 2、新建一个`source`分支，新建md文件初始化`source`分支
- 3、本地拉取`source`分支代码
  - 然后将你本地的之前博客项目的文件都复制过去（除了`.deploy_git`,`.github`,`.git`等文件，themes下的主题中`.github`也删掉）
  - 并新建一个`.gitignore`文件，内容如下
```
.DS_Store
.idea/
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
package-lock.json
```
- 4、把代码推送到`source`分支
- 5、电脑配置git ssh公钥`自行百度`
- 6、安装部署插件`npm install hexo-deployer-git --save`
  - `_config.yml`文件中配置部署:
```yml
# Deployment 部署配置
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git #仓库类型
  repo: git@github.com:AAAJQHAAA/aaajqhaaa.github.io.git #仓库地址
  branch: main #主分支
```
- 7、打包部署，编译后的文件会推送到主分支
  - 清除：`npm run clean`
  - 打包：`npm run build`
  - 部署：`npm run deploy`
- 8、github配置访问
  - `settings`-`pages`-`Source`-选择部署主分支
  - 然后就可以访问了`账号名.github.io`
![012deploy.png](012deploy.png)
