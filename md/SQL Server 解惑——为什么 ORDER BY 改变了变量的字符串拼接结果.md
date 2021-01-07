> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/kerrycode/p/14244499.html)

  在 SQL Server 中可能有这样的拼接字符串需求，需要将查询出来的一列拼接成字符串，如下案例所示，我们需要将 AddressID <=10 的 AddressLine1 拼接起来，分隔符为 |。如下截图所示。这种方式看起来似乎没有什么问题，而且简单测试也是 OK：

```
USE AdventureWorks2014;
```

```
GO
```

```
DECLARE @address_list NVARCHAR(MAX);
```

```
SET @address_list ='';
```

```
SELECT @address_list = @address_list + AddressLine1 + '|' FROM [Person].[Address] WHERE AddressID <=10;
```

```
SELECT @address_list
```

[![](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084337159-202268538.png)](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084336604-605578147.png)

但是, 如果 SQL 多了一个排序操作，结果就变了，这个 SQL 的变量 @address_list 只获取到了最后一条记录”9833 Mt. Dias Blv.|“，

```
USE AdventureWorks2014;
```

```
GO
```

```
DECLARE @address_list NVARCHAR(MAX);
```

```
SET @address_list ='';
```

```
SELECT @address_list = @address_list + AddressLine1 + '|' FROM [Person].[Address] WHERE AddressID <=10 ORDER BY 1;
```

```
SELECT @address_list
```

[![](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084339175-930402026.png)](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084338621-408992928.png)

但是你使用其它一些字段排序的话，它又是 OK 的。在各种实际生产环境中，可能按某个字段排序，字符串拼接就不正常了。但是按有些字段排序又是正常的。有点搞不清套路。下面简单构造一个案例 

```
USE AdventureWorks2014;
```

```
GO
```

```
CREATE TABLE TEST
```

```
(
```

```
    ID        INT NOT NULL
```

```
   ,NAME    NVARCHAR(100) NOT NULL 
```

```
   ,SortID  INT NOT NULL
```

```
   ,CONSTRAINT PK_TEST PRIMARY KEY (ID)
```

```
);
```

```
INSERT INTO dbo.TEST
```

```
SELECT 1, 'Kerry'  , 1 UNION ALL 
```

```
SELECT 2, 'Jerry'  , 2 UNION ALL
```

```
SELECT 3, 'Ken'    , 3 UNION ALL
```

```
SELECT 4, 'Richard', 4 UNION ALL
```

```
SELECT 5, 'Jimmy'  , 5;
```

```
DECLARE @name_list NVARCHAR(100);
```

```
SET @name_list='';
```

```
SELECT @name_list = @name_list + t.NAME + '|'
```

```
FROM dbo.TEST t
```

```
ORDER BY t.SortID;
```

```
SELECT @name_list;
```

上面脚本测试都正常，下面测试就会出现连接字符串只获取了最后一行记录的情况。

```
DECLARE @name_list NVARCHAR(100)='';
```

```
SET @name_list=' '
```

```
SELECT @name_list = @name_list + t.NAME + '| '
```

```
FROM dbo.TEST t
```

```
WHERE ID IN (1,2,3)
```

```
ORDER BY t.SortID;
```

```
SELECT @name_list;
```

[![](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084339964-414882322.png)](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084339577-1846567052.png)

在生产环境还有各种魔幻的现象，按其中一个字段排序是正常，换另外一个字段排序就出现这种现象。如果你将上面测试表的字段的大小修改一下，然后测试下面脚本，发现又不会出现这种情况： 

```
USE AdventureWorks2014;
```

```
GO
```

```
DROP TABLE dbo.TEST;
```

```
GO
```

```
CREATE TABLE TEST
```

```
(
```

```
    ID         INT NOT NULL
```

```
   ,NAME       NVARCHAR(32) NOT NULL 
```

```
   ,SortID     INT NOT NULL
```

```
   ,CONSTRAINT PK_TEST PRIMARY KEY (ID)
```

```
);
```

```
INSERT INTO dbo.TEST
```

```
SELECT 1, 'Kerry'  , 1 UNION ALL 
```

```
SELECT 2, 'Jerry'  , 2 UNION ALL
```

```
SELECT 3, 'Ken'    , 3 UNION ALL
```

```
SELECT 4, 'Richard', 4 UNION ALL
```

```
SELECT 5, 'Jimmy'  , 5;
```

[![](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084340726-951636873.png)](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084340346-323882514.png)

初看像一个 “Bug”，但是它确实不是一个 Bug，官方文档 http://support.microsoft.com/kb/287515 有介绍这个现象，但是目前现在这个链接失效了，搜索也找不到对应的链接了（微软的官方文档这一点是相当坑爹，不如 Oracle 做得好，经常一个链接失效，好的情况是链接换了，糟糕的情况就是这种，根本找不到了），下面的资料是在其它资料里面引用 KB 287515 的内容：

事实证明，**此迭代级联 / 迭代拼接（iterative concatenation）的功能是不受支持的功能。 Microsoft 知识库文章 287515 指出**

   You may encounter unexpected results when you apply any operators or expressions to the ORDER BY clause of aggregate concatenation queries.

   we do not make any guarantees on the correctness of concatenation queries (like using variable assignments with data retrieval in a specific order). The query output can change in SQL Server 2008 depending on the plan choice, data in the tables etc. You shouldn't rely on this working consistently even though the syntax allows you to write a SELECT statement that mixes ordered rows retrieval with variable assignment.

   The correct behavior for an aggregate concatenation query is undefined

       简单来说，这样拼接字符串，虽然在语法上支持，但是却不能保证这样的结果正确性，聚合串联查询的行为是不确定的。如果想安全可靠的拼接字符串的话，有下面一些方式：

1： 使用游标循环循环处理拼接字符串。

2： 使用 XML 查询拼接字符串

方式 1：

```
DECLARE @name_list VARCHAR(512);
```

```
SELECT  @name_list=
```

```
(
```

```
SELECT  t.NAME + '|'
```

```
FROM dbo.TEST t
```

```
WHERE ID IN (1,2,3)
```

```
ORDER BY t.SortID
```

```
FOR XML PATH(''), TYPE
```

```
).value('.', 'varchar(max)')
```

```
SELECT @name_list;
```

方式 2：

```
SELECT Name + '|' AS 'data()' 
```

```
FROM dbo.TEST
```

```
WHERE ID IN (1,2,3)
```

```
FOR XML PATH('');
```

方式 3: 借助 STUFF 函数

注意，使用 COALESCE 有可能也是不行的。如果定义 @name_list 为 VARCHAR(512) 或 VARCHAR(MAX) 则是 OK 的。 

```
DECLARE @name_list VARCHAR(100);
```

```
SELECT @name_list = COALESCE(@name_list + ', ', '') + Name
```

```
FROM dbo.TEST
```

```
WHERE ID IN (1,2,3)
```

```
ORDER BY SortID
```

```
SELECT @name_list
```

[![](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084341490-1185736344.png)](https://img2020.cnblogs.com/blog/73542/202101/73542-20210107084341121-2081691928.png)

3： 使用 CRL 聚合拼接字符串。

4： 如果 SQL Server 2017 使用 STRING_AGG 实现。

```
SELECT  STRING_AGG(Name, '|') AS Departments
```

```
FROM dbo.TEST
```

```
WHERE ID IN (1,2,3)
```

```
SELECT SortID, STRING_AGG(Name, '|') AS Departments
```

```
FROM dbo.TEST
```

```
WHERE ID IN (1,2,3)
```

```
GROUP BY SortID
```

```
ORDER BY SortID;
```

**参考资料：**

https://stackoverflow.com/questions/5538187/why-sql-server-ignores-vaules-in-string-concatenation-when-order-by-clause-speci/5538210#5538210

[https://stackoverflow.com/questions/194852/how-to-concatenate-text-from-multiple-rows-into-a-single-text-string-in-sql-serv](https://stackoverflow.com/questions/194852/how-to-concatenate-text-from-multiple-rows-into-a-single-text-string-in-sql-serv)