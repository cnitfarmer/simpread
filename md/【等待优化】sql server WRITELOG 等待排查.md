> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/gered/p/12449594.html)

**目录**

*   [【0】故障信息](#_label0)
*   [【1】WRITELOG 的分析](#_label1)
*   [【2】解决思路参考](#_label2)
*   [【3】查看参考](#_label3)
*   [也可以看看](#_label4)

[回到顶部](#_labelTop)

【0】故障信息
-------

几乎每个 insert / update 都会有 writelog 等待。

而且我查看了一下，并没有任何阻塞。

　　![](https://img2020.cnblogs.com/i1/1302413/202003/1302413-20200309171813418-725716346.png)

[回到顶部](#_labelTop)

【1】WRITELOG 的分析
---------------

当 SQL Server 会话等待 **WRITELOG 等待类型时**，它将等待将日志缓存的内容写入存储事务日志的磁盘。

为了更详细地说明该过程，假定会话启动一个事务，该事务将执行多个 INSERT 语句。插入数据时，将发生两个操作：

1.  缓冲区高速缓存中的数据页将使用新数据进行更新。
2.  数据被写入日志缓存，这是用于记录将用于回滚事务或写入日志文件的数据的内存段。

此过程将继续进行，直到事务完成（提交）为止，此时日志缓存中的数据立即被写入物理日志文件。当 SQL Server 将日志缓存刷新到磁盘时，会话将以 WRITELOG 等待类型等待。

获取更多信息

如果会话始终在 WRITELOG 等待类型上等待，请查看以下 Perfmon 数据以获取存储事务日志的磁盘瓶颈迹象：

```
物理磁盘对象
　　1. 平均 磁盘队列长度–已排队的 IO 请求的平均数量。如果该值始终大于 1，则可能表明存在磁盘瓶颈。
　　2. 平均 磁盘秒/读取和平均 磁盘秒/写入–如果其中任何一个大于 15-20 毫秒，则可能表示事务日志存储在速度较慢的设备上
SQLServer：缓冲区管理器
　　1. 检查点页面数/秒–需要将所有脏缓冲区写入磁盘的检查点操作刷新的页面数
```

使用 [SolarWinds Database Performance Analyzer](http://www.solarwinds.com/database-performance-analyzer-sql-server.aspx) 或其他工具，还可以确定等待 WRITELOG 事件的顶部 SQL 语句。

如果发现许多语句正在等待，则可能表明上述问题之一是问题所在。如果仅发现一些等待 WRITELOG 的 SQL 语句，则可能表明事务使用效率低下（在下面的示例中进行讨论）。

WRITELOG 等待类型：解决问题

**磁盘子系统性能** –在有关 WRITELOG 等待类型的许多文档中，该问题似乎常常被误解为磁盘子系统问题。在出现磁盘问题的情况下，Perfmon 中的 PhysicalDisk 对象的计数器将很高，并且修复程序通常包括：

1.  向存储事务日志的磁盘子系统添加额外的 IO 带宽。
2.  从磁盘移动非事务日志 IO。
3.  将事务日志移到不太忙的磁盘上。
4.  在某些情况下，减少事务日志的大小也有所帮助。

**频繁提交数据–**在性能咨询期间，在很多情况下，过度热心使用事务可能会导致对 WRITELOG 等待类型的过多等待，即过于频繁地提交数据。为了说明此问题，请考虑以下代码示例：

**示例 1：**以下代码执行了 418 秒，并在 WRITELOG 等待类型上等待了 410 秒。请注意，COMMIT 语句如何位于循环内部并执行了 100,000 次。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
DECLARE @i INT
 SET @i = 1
 WHILE @i < 100000
 BEGIN
 BEGIN TRANSACTION
 INSERT INTO [splendidCRM].[dbo].[product]
 ([productid],
 [category],
 [name],
 [descn])
 VALUES (@i,
 floor(@i / 1000),
 'PROD' + REPLACE(str(@i),' ',''),
 'PROD' + REPLACE(str(@i),' ',''))
 SET @i = @i + 1
 COMMIT
 END
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**示例 2：**以下代码（如上所述也插入了 100,000 行）耗时 3 秒，并且在 WRITELOG 等待类型上等待的时间少于一秒。请注意，COMMIT 语句如何位于循环外部，并且仅执行一次。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
DECLARE @i INT
 SET @i = 1
 BEGIN TRANSACTION
 WHILE @i < 100000
 BEGIN
 INSERT INTO [splendidCRM].[dbo].[product]
 ([productid],
 [category],
 [name],
 [descn])
 VALUES (@i,
 floor(@i / 1000),
 'PROD' + REPLACE(str(@i),' ',''),
 'PROD' + REPLACE(str(@i),' ',''))
 SET @i = @i + 1
 END
 COMMIT
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

最后的想法

当发现会话正在等待 **WRITELOG 等待类型时**，必须对情况进行全面分析，包括检查磁盘性能数据以及查看所有在 WRITELOG 上找到的查询语句。只有这样，您才能放心，您正在解决正确的问题，而不仅仅是添加无法解决根本原因的昂贵硬件。

[回到顶部](#_labelTop)

【2】解决思路参考
---------

有时系统卡住，因此首先尝试清除等待状态，或查看系统日志是否磁盘有问题  
1.dbcc sqlperf('sys.dm_os_wait_stats',clear)  
2. 分离数据库数据文件和日志文件是单独的驱动器。  
3. 如果日志文件大小太大，则限制其大小并插入另一个 LDF。  
4. 如果平均磁盘队列长度大于 1，则它将导致磁盘瓶颈。还需要检查 DISK 突袭等级。

5. 设置恢复模式为简单模式

[回到顶部](#_labelTop)

【3】查看参考
-------

 参考：[https://www.sqlserver-dba.com/2012/01/sql-server-writelog-and-how-to-reduce-it.html](https://www.sqlserver-dba.com/2012/01/sql-server-writelog-and-how-to-reduce-it.html)

SQL Server 联机丛书将等待类型 WRITELOG 定义为 “在等待日志刷新完成时发生。导致日志刷新的常见操作是检查点和事务提交。“

检查点将当前在缓冲区中的所有 [SQL 脏页](https://www.sqlserver-dba.com/2011/11/sql-dirty-pages.html "SQL脏页")写入磁盘。事务提交使数据修改在数据库中永久存在。

注意事项：

　　1）最高优先级–检查磁盘 io susbsystem 性能。

　　2）sys.dm_io_virtual_file_stats DMV 通过返回有关数据库文件的 IO 统计信息来帮助识别性能下降。

　　sys.dm_io_virtual_file_stats 上的 io_stall 很好地指示了 IO 问题。没有确定的数字，但是越低越好。

根据相关工作负载对某些系统进行基准测试，并作为指导。

　　3）如果事务日志与数据共享相同的驱动器，则将其分离到映射到不同 IO 通道的不同驱动器 \ 上。

　　4）相对于工作负载，不仅是缓慢的 IO 子系统，而且可能是导致 IO 子系统过度工作的原因

[回到顶部](#_labelTop)

**也可以看看**
---------

**[计算磁盘 IO 吞吐量和每秒 MB](https://www.sqlserver-dba.com/2011/08/calculate-disk-io-throughput-and-mb-per-second.html "计算磁盘IO吞吐量和每秒MB ")**

**[SQL Server IO 模式和 RAID 级别](https://www.sqlserver-dba.com/2011/06/sql-server-io-patterns-and-raid-levels.html "SQL Server IO模式和RAID级别 ")**

**[磁盘 IO 性能，磁盘块大小调整和 SQL Server](https://www.sqlserver-dba.com/2011/05/disk-io-performance-disk-block-size-tuning-and-sql-servers.html "磁盘IO性能，磁盘块大小调整和SQL Server ")**

  
**作者：Jack Vamvas（[http://www.sqlserver-dba.com](https://www.sqlserver-dba.com/)）**

 

**参考文档：**

 

[Resolving SQL Server Transaction Log Waits](https://academyict.net/2016/06/20/resolving-sql-server-transaction-log-waits/)

[https://logicalread.com/2012/11/08/sql-server-writelog-wait-dr01/#.XmX1AFIzaUk](https://logicalread.com/2012/11/08/sql-server-writelog-wait-dr01/#.XmX1AFIzaUk)