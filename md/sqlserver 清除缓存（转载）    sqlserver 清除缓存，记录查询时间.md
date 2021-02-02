> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/hao-1234-1234/p/8484964.html)

[sqlserver 清除缓存，记录查询时间](http://www.cnblogs.com/50614090/p/4001464.html)
=======================================================================

 

```
--1. 将当前数据库的全部脏页写入磁盘。“脏页”是已输入缓存区高速缓存且已修改但尚未写入磁盘的数据页。
--   CHECKPOINT 可创建一个检查点，在该点保证全部脏页都已写入磁盘，从而在以后的恢复过程中节省时间。
CHECKPOINT
--2. 若要从缓冲池中删除清除缓冲区，请首先使用 CHECKPOINT 生成一个冷缓存。这可以强制将当前数据库的全部脏页写入磁盘，然后清除缓冲区。
--   完成此操作后，便可发出 DBCC DROPCLEANBUFFERS 命令来从缓冲池中删除所有缓冲区。
DBCC DROPCLEANBUFFERS
--3. 释放过程缓存将导致系统重新编译某些语句（例如，即席 SQL 语句），而不重用缓存中的语句。
DBCC FREEPROCCACHE
--4. 从所有缓存中释放所有未使用的缓存条目。SQL Server 2005 Database Engine 会事先在后台清理未使用的缓存条目，以使内存可用于当前条目。
--  但是，可以使用此命令从所有缓存中手动删除未使用的条目。
DBCC FREESYSTEMCACHE ( 'ALL' )
--5. 要接着执行你的查询,不然SQLServer会时刻的自动往缓存里读入最有可能需要的数据页.
```

```
CHECKPOINT;
DBCC DROPCLEANBUFFERS;
DBCC FREEPROCCACHE;
DBCC FREESYSTEMCACHE ('ALL');
SET STATISTICS TIME ON ;
--查询条件
SET STATISTICS TIME OFF;
```

转载来源：https://www.cnblogs.com/50614090/p/4001464.html

```
--1. 将当前数据库的全部脏页写入磁盘。“脏页”是已输入缓存区高速缓存且已修改但尚未写入磁盘的数据页。
--   CHECKPOINT 可创建一个检查点，在该点保证全部脏页都已写入磁盘，从而在以后的恢复过程中节省时间。
CHECKPOINT
--2. 若要从缓冲池中删除清除缓冲区，请首先使用 CHECKPOINT 生成一个冷缓存。这可以强制将当前数据库的全部脏页写入磁盘，然后清除缓冲区。
--   完成此操作后，便可发出 DBCC DROPCLEANBUFFERS 命令来从缓冲池中删除所有缓冲区。
DBCC DROPCLEANBUFFERS
--3. 释放过程缓存将导致系统重新编译某些语句（例如，即席 SQL 语句），而不重用缓存中的语句。
DBCC FREEPROCCACHE
--4. 从所有缓存中释放所有未使用的缓存条目。SQL Server 2005 Database Engine 会事先在后台清理未使用的缓存条目，以使内存可用于当前条目。
--  但是，可以使用此命令从所有缓存中手动删除未使用的条目。
DBCC FREESYSTEMCACHE ( 'ALL' )
--5. 要接着执行你的查询,不然SQLServer会时刻的自动往缓存里读入最有可能需要的数据页.
```

```
CHECKPOINT;
DBCC DROPCLEANBUFFERS;
DBCC FREEPROCCACHE;
DBCC FREESYSTEMCACHE ('ALL');
SET STATISTICS TIME ON ;
--查询条件
SET STATISTICS TIME OFF;
```

转载来源：https://www.cnblogs.com/50614090/p/4001464.html