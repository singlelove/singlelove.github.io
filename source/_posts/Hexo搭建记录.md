---
title: Hexo搭建记录
date: 2018-03-06 10:02:53
tags: 学习
categories: 框架
---
# 前言
本文是我在Hexo搭建、主题配置及插件安装过程中的记录，旨在帮助那些同样想学习使用Hexo搭建自己个人站点的朋友。不过由于本文是在我搭建Hexo一个多月之后才开始写作的，因此难免存在一些纰漏之处，欢迎指正。

# 准备工作
## 1.安装git
- [下载地址](https://git-scm.com/downloads)
- 安装：一路next，安装完成之后，打开git bash，输入git version显示版本号表示安装成功

## 2.安装nodejs
- [下载地址](http://nodejs.cn/)
- 安装：一路next，安装完成之后，打开git bash，输入node -v显示版本号表示安装成功

# 部署到github
- 没账号的创建账号，然后创建一个repo，名称为`yourname.github.io`，其中yourname(以及之后出现的yourname)是你的github名称。
{% qnimg Hexo搭建记录/createRepo.png %}
{% qnimg Hexo搭建记录/repoConfig.png %}
- 添加SSH，参考github[官网文档](https://help.github.com/articles/connecting-to-github-with-ssh/)
- 搭建开发环境
  1、创建hexo分支
  2、设置hexo为默认分支
  3、在git bash中使用git clone git@github.com:yourname/yourname.github.io.git拷贝仓库到本地
  4、在本地yourname.github.io文件夹下通过git bash执行以下命令安装hexo及相关依赖
> npm install hexo-cli -g
hexo init
npm install
npm install hexo-deployer-git --save

  5、修改_config.yml中的deploy参数，分支为master
  6、执行以下命令将改动提交到github
> git add .
git commit -m "..."
git push origin hexo
  
  7、执行以下命令可进行本地开发，打开localhost:4000即可浏览
> hexo g
hexo s

  8、执行以下命令可部署到github，打开yourname.github.io即可浏览
> hexo g
hexo d

  至此，yourname.github.io仓库上有两个分支，hexo分支用来存放网站原始文件，master分支用来存放生成的静态文件，这样即使换了电脑，也可以通过上述3、4两步快速搭建本地开发环境(此时，不需要hexo init)。
  
# 主题安装
主流是NexT主题，安装没什么难度，可以参考[官方文档](http://theme-next.iissnan.com/third-party-services.html#livere)

# 插件安装
- 评论系统——[来必力](http://theme-next.iissnan.com/third-party-services.html#livere)
- 访问数统计——[不蒜子](http://theme-next.iissnan.com/third-party-services.html#analytics-busuanzi)
- 阅读次数统计——[LeanCloud](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
- 搜索服务——[Algolia](http://theme-next.iissnan.com/third-party-services.html#algolia-search)

# 发表文章
## 新建文章
在git bash中通过以下命令新建文章
> hexo n "articleTitle"

然后在source/_posts 目录中就可找到"articleTitle.md"文件，之后可通过Markdown编辑器打开进行编辑，作者是用的是[CmdMarkdown](https://www.zybuluo.com/mdeditor)
> CmdMarkdown似乎只支持在线编辑，无法保存到本地，因此需要编辑完成后复制到本地文件里。此外由于不同markdown编辑器的渲染方式不同，因此用编辑器看到的效果与Hexo最终渲染的效果不尽相同。

## 插入图片
插入图片有两种方式，一种是将图片放在项目的资源文件中，直接部署到github上；一种是图床上，通过外链引入。
第一种方式在图片不多的情况下可以使用，比较方便，配置参考[官方文档](https://hexo.io/zh-cn/docs/asset-folders.html)
第二种方式推荐选用[七牛](https://www.qiniu.com/)作为图床，相关配置可参考文章["使用七牛为Hexo存储图片等资源"](https://yuchen-lea.github.io/2016-01-21-use-qiniu-store-file-for-hexo/)
> 使用hexo-qiniu-sync插件时，请谨慎执行hexo clean命令，因为会清空资源目录。