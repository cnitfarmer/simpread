\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/ljhdo/p/13901665.html)

实时运营数据分析（real-time operational analytics ）是指同时在同一张数据表上执行分析处理和业务处理。分析查询主要是对海量数据执行聚合查询，而事务主要是指对数据表进行少量数据的更新和查找。

运营工作负载（Operational workload）是指对开展业务至关重要的业务交易。例如，一家零售商店有一个交易系统来创建或修改新订单，而一家信用卡公司则跟踪供应商代表其客户收取的所有费用。 这些交易系统对企业至关重要，因为任何停机时间或速度放缓都会直接影响企业的利润。 因此，这些系统专为性能 / 可伸缩性而设计，并具有高可用性。 对业务工作负载同样重要的是企业用来回答诸如 “完成订单的平均时间是多少时间” 之类的问题的分析。

大多数客户通过在另一台服务器上创建数据仓库来实现分析工作负载（analytics workload），周期性把交易系统产生的数据通过 ETL（提取，转换和加载）同步到数据仓库。

一，传统的 BI 系统
-----------

在传统的 BI 系统中，把用于公司日常事务的工作负载（OLTP）和用于报表分析的工作负载分开，分别对应于运营数据库和数据仓库，通过把两种工作负载分开，尽可能地避免分析查询对交易数据库的影响。

分析数据通常存储在专用于用于报表分析查询的数据仓库或数据集市中。日常的事务数据直接写入到运营数据库，为了使分析尽量准备准确，需要数据工程师设计 ETL（提取，转换和加载）程序，定期把运营数据转移到分析数据库（即数据仓库）中。

![](https://img2020.cnblogs.com/blog/628084/202010/628084-20201030175343607-384832405.png)

尽管有 ETL 程序定期从事务数据库把数据同步到分析数据库，但是数据的同步不可避免地存在一定的时延，这就导致分析数据库中的数据还是跟运营数据库存在一定的差异（GAP），使得数据分析的结果存在失真的可能性。

二，什么是实时运营数据分析
-------------

传统的 BI 系统存在数据延迟的原因是用于分析处理和事务处理的表是分开的，如果分析处理和事务处理在同一基础表上执行，那么就不会存在时间延迟的问题。

实时运营分析的目标是单个数据源的场景，例如企业资源计划（ERP）应用程序，用户可以在其上执行日常的事务处理和分析工作。

实时分析在行存储表上使用可更新的非聚集列存储索引，列存储索引维护数据的副本，分别在数据的单独副本上运行事务处理和分析工作，这样避免了在同一个数据表运行两个工作负载，可以最大程度地减少同时运行的两个工作负载对查询性能的影响。

![](https://img2020.cnblogs.com/blog/628084/202010/628084-20201030173113667-1940723065.png)

当基表的数据更新时，SQL Server 自动更新列存储索引，通过这种设计，用户可以对最新数据实时运行分析，分析查询使用的始终是最新的数据，分析处理的结果是最接近现实数据的，因此，得到的分析结果是最真实和最新的。

用户只需要在基表上创建一个非聚集的列存储索引，就可以实现实时运营数据分析：

```
CREATE NONCLUSTERED COLUMNSTORE INDEX index\_name
ON table\_name (colum\_list) ;
```

在表上创建非聚集的列存储索引，底层的物理结构如下图所示：

![](https://img2020.cnblogs.com/blog/628084/202010/628084-20201030181758062-298380547.jpg)

对运营数据进行实时分析，存在两个挑战：

第一个挑战是，当数据库模式已针对 OLTP 优化时，如何获得良好的分析查询性能？传统的数据仓库（DW）使用星型或雪花模式为分析查询提供最佳性能。但是，OLTP 数据库的架构已高度规范化（即数据重复最少），这主要是因为大量表之间的连接复杂性，在用于分析查询时可能会导致性能不佳。列存储索引（即 NCCI）可以显着提高查询速度，可以克服查询的复杂性，并在几秒钟内仍然可以提供大多数分析查询。在这一点上，必须强调的是，使用实时操作分析的分析查询性能不会像使用专用 DW 时所能达到的那样快，但是关键好处是能够进行实时分析。一些企业可能选择进行实时运营分析，同时仍保留用于极端分析的专用 DW。有些客户已经在生产中部署了 NCCI，并取消了专用 DW。

第二个挑战是，如何最小化或消除分析对事务性工作负载的影响？实际上，交易系统会跟踪其业务交易，交易工作量的任何减缓都是不可接受的。虽然 SQL Server 提供了多个选项来最小化分析对运营工作负载的影响，但是这种影响是不可避免的。

参考文档：

[Get started with Columnstore for real-time operational analytics](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics?view=sql-server-ver15)

[Real-Time Operational Analytics Using In-Memory Technology](https://cloudblogs.microsoft.com/sqlserver/2015/12/09/real-time-operational-analytics-using-in-memory-technology/)

[Real-Time Operational Analytics - Overview nonclustered columnstore index (NCCI)](https://docs.microsoft.com/en-us/archive/blogs/sqlserverstorageengine/real-time-operational-analytics-using-nonclustered-columnstore-index)