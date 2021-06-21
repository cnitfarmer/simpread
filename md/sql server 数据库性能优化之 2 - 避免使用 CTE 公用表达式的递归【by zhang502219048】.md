> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zhang502219048/p/14911013.html)

数据库优化中的一个实例，记录一下：  
1. 原来用了 CTE 公用表达式的递归，reads 高达约 40 万，看查询执行计划，使用了 Nested Loops；  
2. 优化去掉递归，改用其它方式实现，reads 降低到 2639，看查询执行计划，避免了使用 Nested Loops. 

 ![](https://img2020.cnblogs.com/blog/1703141/202106/1703141-20210621083052690-1487330256.png)

欢迎转载，但转载请务必注明博文来源和作者！  
* 来源：https://www.cnblogs.com/zhang502219048/p/14911013.html  
* 作者博客园博主：zhang502219048  
* 微信公众号名称：SQL 数据库编程  
* 微信号：zhang502219048