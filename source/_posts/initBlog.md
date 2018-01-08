---
title: 个人博客搭建
date: 2017-12-31 19:55:25
categories:
- 杂项
---

*关于我如何搭建了自己的个人博客，以及遇到的一些奇怪的问题。*
*resources: [hexo中文官网](https://hexo.io/zh-cn/) & [next主题中文官网](http://theme-next.iissnan.com/getting-started.html) *
<!--more-->

# 前情提要

* Github Pages + Hexo

# 搭建Hexo

## Nodejs
* 建议直接[官网](https://nodejs.org/zh-cn/)，上面有足够多和清晰的下载资源
* 官网慢的话，可以看看自己公司内网的资源，一般前端都会有一个公共的资源库

## Hexo
* 按照步骤，确保已经完成nodejs的安装
* 执行以下命令安装，如果npm较慢，可以指定国内资源方，一般公司也有
```
npm install hexo-cli -g [--registry=http://*]
hexo init <folder> #新建一个文件夹，作为你的博客目录, 以下以MyBlog代指
cd MyBlog
npm install
hexo server #启动本地服务器, 默认端口4000, 因此可以在(localhost:4000)预览
```

## Theme
>在 Hexo 中有两份主要的配置文件，其名称都是 _config.yml。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。
>为了描述方便，在以下说明中，将前者称为 站点配置文件， 后者称为 主题配置文件。

* 主题在[官网](https://hexo.io/themes/)就有很多选择
* 我懒，就使用了最常见的[Next](http://theme-next.iissnan.com/)
* 直接git clone下来某个主题，并放到MyBlog/themes目录下
* 在站点配置文件中找到themes关键字，并修改为next
```
#克隆最新版本
cd MyBlog  #进入我的博客目录
git clone https://github.com/theme-next/hexo-theme-next.git themes/next

#修改主题配置文件
## Themes: https://hexo.io/themes/
theme: next
```

# Github Pages

* 安装插件
```
npm install hexo-deployer-git --save
```

* 修改站点配置文件，将功能部署在master分支
```
deploy:
  type: git
  repo: https://github.com/ding-guohang/ding-guohang.github.io.git
  branch: master
```

* 建议将源码保留在另一个分支，例如code
···
git init
git checkout -b code
git add .
git commit -m "init code branch for codes"
git push --set-upstream origin code
···


# 绑定域名

* 我是学前辈在[Godaddy](https://www.godaddy.com/)上购买的域名
* 修改Github Pages对应的Repository的Setting，有填写绑定域名的地方
* 修改域名的DNS配置
** 新增/修改一条Type为A的记录，使其Name为@，value为192.30.252.153
** 新增/修改一条Type为CNAME的记录，使其Name为WWW，value为你的Github Pages的地址


# 博客更新

* 在code分支上进行更新
```
hexo generate (hexo g) 
hexo deploy (hexo d)
```
