> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/MrHSR/p/9287226.html)

### 一. 概念

　　在介绍资源等待 PAGEIOLATCH 之前，先来了解下从实例级别来分析的各种资源等待的 dmv 视图 sys.dm_os_wait_stats。它是返回执行的线程所遇到的所有等待的相关信息，该视图是从一个实际级别来分析的各种等待, 它包括 200 多种类型的等待，需要关注的包括 PageIoLatch（磁盘 I/O 读写的等待时间）,LCK_xx（锁的等待时间），WriteLog（日志写入等待），PageLatch（页上闩锁）Cxpacket（并行等待）等以及其它资源等待排前的。 [  
](https://www.cnblogs.com/MrHSR/p/9278270.html)

　　1.  下面根据总耗时排序来观察，这里分析的等待的 wait_type 不包括以下

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
SELECT  wait_type ,
        waiting_tasks_count,
        signal_wait_time_ms ,
        wait_time_ms,
        max_wait_time_ms
FROM    sys.dm_os_wait_stats
WHERE   wait_time_ms > 0
        AND wait_type NOT IN ( 'CLR_SEMAPHORE', 'CLR_AUTO_EVENT',
                               'LAZYWRITER_SLEEP', 'RESOURCE_QUEUE',
                               'SLEEP_TASK', 'SLEEP_SYSTEMTASK',
                               'SQLTRACE_BUFFER_FLUSH', 'WAITFOR',
                               'LOGMGR_QUEUE', 'CHECKPOINT_QUEUE',
                               'REQUEST_FOR_DEADLOCK_SEARCH', 'XE_TIMER_EVENT',
                               'BROKER_TO_FLUSH', 'BROKER_TASK_STOP',
                               'CLR_MANUAL_EVENT',
                               'DISPATCHER_QUEUE_SEMAPHORE',
                               'FT_IFTS_SCHEDULER_IDLE_WAIT',
                               'XE_DISPATCHER_WAIT', 'XE_DISPATCHER_JOIN',
                               'SQLTRACE_INCREMENTAL_FLUSH_SLEEP' )
ORDER BY signal_wait_time_ms DESC
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

  下图排名在前的资源等待是重点需要去关注分析：

![](https://images2018.cnblogs.com/blog/151560/201807/151560-20180709102928221-936631197.png)

　　通过上面的查询就能找到 PAGEIOLATCH_x 类型的资源等待，由于是实例级别的统计，想要获得有意义数据，就需要查看感兴趣的时间间隔。如果要间隔来分析，不需要重启服务，可通过以下命令来重置

```
DBCC SQLPERF ('sys.dm_os_wait_stats', CLEAR);
```

　　wait_type: 等待类型  
　　waiting_tasks_count: 该等待类型的等待数  
　　wait_time_ms: 该等待类型的总等待时间 (包括一个进程悬挂状态 (Suspend) 和可运行状态 (Runnable) 花费的总时间)  
　　max_wait_time_ms: 该等待类型的最长等待时间  
　　signal_wait_time_ms: 正在等待的线程从收到信号通知到其开始运行之间的时差 (一个进程可运行状态 (Runnable) 花费的总时间)  
　　io 等待时间 ==wait_time_ms - signal_wait_time_ms

### 二. PAGEIOLATCH_x

　　2.1 什么是 Latch

　　　　在 sql server 里 latch 是轻量级锁，不同于 lock。latch 是用来同步 sqlserver 的内部对象 (同步资源访问)，而 lock 是用来对于用户对象包括 (表，行，索引等) 进行同步，简单概括：Latch 用来保护 SQL server 内部的一些资源（如 page）的物理访问，可以认为是一个同步对象。而 lock 则强调逻辑访问。比如一个 table，就是个逻辑上的概念。关于 lock 锁这块在 "[sql server 锁与事务拨云见日](http://www.cnblogs.com/MrHSR/p/9119107.html) " 中有详细说明。

　　2.2 什么是 PageIOLatch　

　　当查询的数据页如果在 Buffer pool 里找到了，则没有任何等待。否则就会发出一个异步 io 操作，将页面读入到 buffer pool, 没做完之前，连接会保持在 PageIoLatch_ex(写) 或 PageIoLatch_sh(读) 的等待状态，是 Buffer pool 与磁盘之间的等待。它反映了查询磁盘 i/o 读写的等待时间。  
　　当 sql server 将数据页面从数据文件里读入内存时，为了防止其他用户对内存里的同一个数据页面进行访问，sql server 会在内存的数据页同上加一个排它锁 latch, 而当任务要读取缓存在内存里的页面时，会申请一个共享锁，像是 lock 一样，latch 也会出现阻塞，根据不同的等待资源，等待状态有如下：PAGEIOLATCH_DT，PAGEIOLATCH_EX，PAGEIOLATCH_KP，PAGEIOLATCH_SH，PAGEIOLATCH_UP。重点关注 PAGEIOLATCH_EX（写入）和 PAGEIOLATCH_SH(读取) 二种等待。

2.1  AGEIOLATCH 流程图

　　有时我们分析当前活动用户状态下时，一个有趣的现象是，有时候你发现某个 SPID 被自己阻塞住了 (通过 sys.sysprocesses 了查看) 为什么会自己等待自己呢？ 这个得从 SQL server 读取页的过程说起。SQL server 从磁盘读取一个 page 的过程如下：

![](https://images2018.cnblogs.com/blog/151560/201807/151560-20180709105611318-776185114.png)

![](https://images2018.cnblogs.com/blog/151560/201807/151560-20180708131628743-1687014755.png)

　　(1)：由一个用户请求，获取扫描 X 表, 由 Worker x 去执行。

　　(2)：在扫描过程中找到了它需要的数据页同 1:100。

　　(3)：发面页面 1:100 并不在内存中的数据缓存里。

　　(4)：sql server 在缓冲池里找到一个可以存放的页面空间，在上面加 EX 的 LATCH 锁，防止数据从磁盘里读出来之前，别人也来读取或修改这个页面。

　　(5)：worker x 发起一个异步 i/o 请求, 要求从数据文件里读出页面 1:100。

　　(6)：由于是异步 i/o(可以理解为一个 task 子线程)，worker x 可以接着做它下面要做的事情，就是读出内存中的页面 1:100, 读取的动作需要申请一个 sh 的 latch。

　　(7)：由于 worker x 之前申请了一个 EX 的 LATCH 锁还没有释放，所以这个 sh 的 latch 将被阻塞住，worker x 被自己阻塞住了，等待的资源就是 PAGEIOLATCH_SH。

　　最后当异步 i/o 结束后，系统会通知 worker x，你要的数据已经写入内存了。接着 EX 的 LATCH 锁释放，worker x 申请得到了 sh 的 latch 锁。

总结：首先说 worker 是一个执行单元, 下面有多个 task 关联 Worker 上， task 是运行的最小任务单元，可以这么理解 worker 产生了第一个 x 的 task 任务，再第 5 步发起一个异步 i/o 请求是第二个 task 任务。二个 task 属于一个 worker，worker x 被自己阻塞住了。 关于任务调度了解查看 [sql server 任务调度与 CPU](https://www.cnblogs.com/MrHSR/p/9087686.html)。

 2.2 具体分析

　　通过上面了解到如果磁盘的速度不能满足 sql server 的需要，它就会成为一个瓶颈，通常 PAGEIOLATCH_SH 从磁盘读数据到内存，如果内存不够大，当有内存压力时候它会释放掉缓存数据，数据页就不会在内存的数据缓存里, 这样内存问题就导致了磁盘的瓶颈。PAGEIOLATCH_EX 是写入数据，这一般是磁盘的写入速度明显跟不上，与内存没有直接关系。

下面是查询 PAGEIOLATCH_x 的资源等待时间：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
select wait_type,
waiting_tasks_count,
wait_time_ms ,
max_wait_time_ms,
signal_wait_time_ms
from sys.dm_os_wait_stats
where wait_type like 'PAGEIOLATCH%' 
order by wait_type
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

下面是查询出来的等待信息：

PageIOLatch_SH 总等待时间是 (7166603.0-15891)/1000.0/60.0=119.17 分钟，平均耗时是 (7166603.0-15891)/297813.0=24.01 毫秒, 最大等待时间是 3159 秒。

PageIOLatch_EX 总等待时间是 (3002776.0-5727)/1000.0/60.0=49.95 分钟，    平均耗时是 (3002776.0-5727)/317143.0=9.45 毫秒，最大等待时间是 1915 秒。

![](https://images2018.cnblogs.com/blog/151560/201807/151560-20180709102631859-2110141968.png)

关于 I/O 磁盘 sys.dm_io_virtual_file_stats 函数也做个参考

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
SELECT  
       MAX(io_stall_read_ms) AS read_ms,
         MAX(num_of_reads) AS read_count,
       MAX(io_stall_read_ms) / MAX(num_of_reads) AS 'Avg Read ms',
         MAX(io_stall_write_ms) AS write_ms,
        MAX(num_of_writes) AS write_count,
         MAX(io_stall_write_ms) /  MAX(num_of_writes) AS 'Avg Write ms'
FROM    sys.dm_io_virtual_file_stats(null, null)
WHERE   num_of_reads > 0 AND num_of_writes > 0
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

![](https://images2018.cnblogs.com/blog/151560/201807/151560-20180710102343877-362515604.png)

　　总结：**PageIOLatch_EX(写入) 跟磁盘的写入速度有关系。PageIOLatch_SH(读取) 跟内存中的数据缓存有关系。**通过上面的 sql 统计查询，从等待的时间上看，并没有清晰的评估磁盘性能的标准，但可以做评估基准数据，定期重置，做性能分析。要确定磁盘的压力，还需要从 windows 系统性能监视器方面来分析。 关于内存原理查看”[sql server 内存初探](http://www.cnblogs.com/MrHSR/p/9073411.html) “磁盘查看 "[sql server I/O 硬盘交互](https://www.cnblogs.com/MrHSR/p/9102479.html) "。