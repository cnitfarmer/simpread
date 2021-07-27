> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44594203/article/details/105331260)

最近在安装 mysql5.6 的时候遇到很多坑。特别是 yum install mysql-server 的时候，下载的总是 mysql8，造成诸多问题。最后发现是因为我是用的是 centos8，centos8 支持的 mysql 版本是 8.0。所以在网上看了一下，有人总结了许多，比如：Centos8 和 7 的区别（参照 redhat） https://www.cnblogs.com/iwalkman/p/11781234.html  
最后，还是在阿里服务器控制台换回了熟悉的 centos7。  
主要我关注的是以下内容，其他的区别知道就行：  
![](https://img-blog.csdnimg.cn/20200405184751260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_6Lev6L656Lev,size_16,color_FFFFFF,t_70)