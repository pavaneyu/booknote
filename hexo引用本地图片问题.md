---
title: Hexo本地图片引用问题
date: 2022年2月14日18:20:45
tags: [博客]
categories: 解决问题记录
---

记录一上午搞个人博客的图片问题

<!--more-->

0.以前用简书写md,导出的md图片不能正常访问,估计简书防盗链拒绝了其他地方访问其服务器上的图片.

1.我对图床很排斥,以为用个图片还得先上传到网络,然后拿着url在md文件中引用,太不优雅了

2.既然md文件不能存储图片信息,那么我把图片跟md文件放在一个文件夹下不就可以

3.hexo 不能把md附带的文件generate到html  关联

4.又是下插件,又是搞配置  ,图片是跟html在一起了,结果html对图片的引用是md文件中的链接 原封不动

关键typora解析md中图片地址是 相对于md文件本身的

html中  http://localhost:4000/111/image-20220214100545449.png 是在public目录下的啊

5.看来只有更改html中对图片的引用地址了,但是hexo中哪段代码实现了这个我哪里去找啊



转机

6.typora设置里有图片上传的功能,查了一下确实可以 粘贴自动把图片上传到云端,并自动引用其地址,

类似简书

<img src="https://s2.loli.net/2022/02/14/LYlnA417sU9E2kv.png" alt="image-20220214121843134" style="zoom: 80%;" />

7.但是要配置2个参数,还要设置一个代理上传到图床的软件,配置好图床token,想用github当图床的

结果半天没成功,选了个最简单的sm:ms当图床,5GB先用着吧,用几年应该没问题