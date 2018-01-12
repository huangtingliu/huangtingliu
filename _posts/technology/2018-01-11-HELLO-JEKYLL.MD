---
layout: default
title: Hello Jekyll
tags: [blog]
imgPath: 2018-01-11-HELLO-JEKYLL
---

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)  
最近突然心血来潮。做了个博客,就是你现在看到的这个。第一次写博客...

## JEKYLL
[Jekyll](https://jekyllrb.com/)基于ruby，用于构建静态网站，有一些模板可以使用、使用liquid代码构建页面静态内容，github page也支持jekyll进行构建博客，在本地安装jekyll，通过jekyll serve --safe会在项目目录下多一个_site目录，里面放一些静态文件，通过浏览器http://localhost:4000xxx.html
可以查看页面，进行调试页面，再将内容上传github项目，发布博客文章。值得注意的是jekyll并不能处理动态请求。

>英文文档:<https://jekyllrb.com/>  
中文文档:<http://jekyllcn.com/>

### 在github上创建jekyll博客项目
在搭建Jekyll博客之前，你需要拥有一个github账号。
这里就不介绍github注册流程了(本文前提你已掌握基本的github操作)。

1. 登录github,创建一个仓库
![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/create-reponsitory.png)
2. 打开项目的setting,拉到GitHub Pages项，选择模板和自定义域名（大概需要十分钟生效），我这里选择的是jekyll-theme-cayman模板
![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/github-setting-github-pages.png)

3. 访问https://(username).github.io/(username)/ 就可以访问了，如果你上一步中配置了Custom domain并对域名做了解析，那么访问你的域名也是可以的。
![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/blog-visit.png)

基础模板有了，那么如何搭建自己需要的效果的博客呢？通过git拉取项目，本地搭建jekyll服务，进行本地调试博客。

### windows本地搭建jekyll服务

1. 下载安装Ruby(<https://rubyinstaller.org/downloads/>)，下载完exe点击安装就可以了
![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/ruby-download.png)

2. 下载安装RubyDevKit,exe文件直接安装，安装完后windows命令行cd xxx进入安装目录，输入dk.rb init, 将会生成一个config.yml文件，最后执行dk.rb install
![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/RubyDevKit-download.png)

3. 下载安装Rubygems,解压后，cmd命令进入安装目录执行命令setup.rb
	>下载地址：<https://rubygems.org/pages/download> 

	gem是有类似npm的自动下载安装依赖包的功能，注意国内的网站有些外国资源访问不了，建议使用淘宝镜像。
>使用方法参考<https://ruby.taobao.org/>

	命令行进入gem安装目录输入：
```
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/  
gem sources -l   
gem install rails  
```
4. 安装jekyll,进入gem安装目录，输入：
```
gem install jekyll
```
5. 测试测试jekyll服务
随便找个地方创建一个文件夹（项目名），cmd命令行进入该目录，输入
```
jekyll serve --safe
```
将会生成一个_site文件夹（如果项目目录有文件，将会全部拷贝一份到_site文件夹下），浏览器访问<http://localhost:4000/>相当于访问了_site文件夹,所以以后你写的文章也会被生成到_site目录下。  

	>命令行

	![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/jekyll-run-cmd.png)
	>生成的目录

	![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/jekyll-run-dir.png)
	>浏览器访问
	
	![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/jekyll-run-visit.png)

>有很多细节的东西可能没讲清楚，希望能帮助到想使用jekyll搭建博客的人

