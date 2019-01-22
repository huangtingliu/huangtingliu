---
layout: default
title: 使用tortoiseSVN将本地文件夹上传至svn
tags: [svn,版本控制]
imgPath: 2019-01-17-TortoiseSVN-IMPORT
---
svn作为常用的版本控制工具拥有大量的用户群体。本文介绍如何将本地文件夹上传至svn。

> 本文教程的前提默认你已经安装好了tortoiseSVN工具，并且拥有svn服务器地址。

假设我们有这样一个文件夹，如何将它上传至svn上？  

![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/1start-dir.png)  

## 一、import
在该本地文件夹目录内 右键 - > tortoiseSVN - > import ,选择要上传的目录（注意最后一个/ 后面的路径就是本地文件夹的名称）  
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/2menu-import.png)
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/3import.png)

执行完后其实该文件夹已经上传到svn上了，但是却没有跟本地文件夹关联起来（关联起来的文件夹会有个勾的图标）。

## 二、chekout
还是在该本地文件夹目录内 右键 - > tortoiseSVN - >执行chekout
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/4menu-checkout.png)
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/5checkout.png)  
图片中2跟3的地址是一样的，1的地址就是刚才import时的地址。  

这个时候会提示不是空文件夹，不管它，直接点checkout即可
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/6checkout-tip.png)  

可以看到文件上已经打上了svn的标记，表示已经与svn服务器关联上了
![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/7finish.png)  