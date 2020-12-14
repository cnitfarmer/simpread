> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/l1pe1/p/7920204.html)

Windows 常用性能计数器总结

基础监控：

1.SQL Server Buffer: Buffer Cache Hit Ratio

这是一个很重要查看内存是否不足的参数。SQL Server Buffer 中的计数器 Buffer Cache Hit Ratio 用来指出 SQLServer 从缓存中而不是磁盘中获得数据的频率。sqlserver 会将某些查询过的数据缓存在内存中用于以后再次查询使用。当一个查询 A 进来了以后数据库会编译这个 sql 看看需要哪些数据，然后执行计划首先去内存中找看是否有这次查询所需要的数据，如果这个同样的 sql 刚才已经执行过了或者该表的数据已经缓存在内存中，但是却没有在内存中找到数据，那就有可能是因为内存不足引起内存挤压将缓存数据写回硬盘或者释放掉来提供数据库其他请求来使用。一般来说 oltp 的系统，这个值最起码也应该在 90% 以上，理想值是 99%。如果这个值低于 90%，那建议你应该添加内存了。

2.Memory: Pages/sec

这个也是监控内存是否不足的一个比较重要的参数。这个计数器记录的是每秒钟内存和磁盘之间交换的页面数。频繁的交换页面就会消耗更多的 io，这会影响到服务器的性能。打个比方，超市有一个货架上边摆满了新进的各种商品 a、b、c，当你去超市想买 a 的时候直接去货架就能拿到 a，方便的很，当顾客进超市逛一圈以后跟你说我怎么没有发现旧商品 d 呢，我就想买这个 d，然后工作人员就会去仓库把商品 d 拿出来摆放到货架上供下次顾客来买。但是货架摆满了怎么办呢，只能将时间长没有人问津的 a 下架放到仓库然后空出来地方摆放 d，但是下次另一个顾客来了又有想要购买 a 的意向，工作人员就得再次把 a 拿出来替换掉货架上的 d。其实内存就是这个货架，硬盘就是仓库。因为货架太小了，导致只能频繁的更换货架上的商品来提供正常的运营，想减少反复来回搬运产生的 io 开销，只能换个更大的货架来满足需求。

如果服务器上只跑的 sqlserver，那这个指标的理想范围应该是 0-20 之间，偶尔超过 20 的话影响不大，如果这个值频繁的超过 20，那说明你的这台服务器可能需要加内存了。

当然这个指标要配合着上一个指标 Buffer Cache Hit Ratio 来看，如果上一个指标缓冲命中一直在 99% 或者更高，而这个期间内你的页交换一直在 20 以上，那意味着不仅仅是内存不足，而且其他的程序占用了系统内存。

3.Memory: Available Bytes

另一个监控内存情况的计数器就是这个。这个值最少最少也得大于 5M，因为 sqlserver 需要始终维持 5-10m 的自由内存用于分配，当这个值低于 5m 的时候，那 sqlserver 可能会因为缺少内存而产生性能瓶颈。

4.Physical Disk: % Disk Time

这个计数器记录的是磁盘的繁忙程度（是整个磁盘阵列或者物理磁盘的繁忙程度）。理论上这个值应该低于 55%，如果持续的高于 55%，那说明这台服务器上可能有 io 瓶颈。

如果只是偶尔的出现几次，那不必担心，但是可以对应的找到这个时间点，数据库正在干嘛执行了哪些语句，对应的优化一下。

5.Physical Disk: Avg. Disk Queue Length

这是一个比较重要的查看磁盘 io 情况的指标。理论上每个物理磁盘的值不应该超过 2。当然这个值是需要计算的，比如用 4 块物理盘做了个 raid10，此时在一个监控周期内磁盘队列的均值是 10，那每块磁盘的队列值就是 10/4=2.5，那么就可以说这个磁盘阵列存在 i/o 瓶颈了。这个跟之前的 disktime 指标一样，偶尔出现不必担心，如果长时间出现，那就得着手考虑解决磁盘的 io 性能问题了。

6.Processor: % Processor Time

这是监控 cpu 情况的一个指标（类似于 disk time）。这个是观察 cpu 利用率的一个关键参数。如果 Processor Time 计数器的值持续超过 80%，说明 cpu 存在瓶颈问题。如果只是偶尔出现，那说明可能是这个时间点有个特别消耗 cpu 的查询，可以在下一次这个时间点来临的时候尝试抓一下 sql 并且优化它。如果在某一个时间点以后 cpu 一直飙高，常见的情况就是：1. 突然间的高并发 2. 索引重整 3. 突然一个经常使用的数据量特别大的索引失效了 4. 死锁 5. 其他好多好多。先找到问题所在，在处理掉它。

7.System: Processor Queue Length

这个指标类似于 disk queue length，也是算单个 cpu 的。单个 cpu 不能超过 2，比如你是 2u 的机器，那这个值不应该超过 4，如果在一个监控周期内持续性的超过 4，那就可能出现 cpu 瓶颈了。

8.Connections Established 当前连接数（Established + Close-Wait）

9.Network Interface:Bytes Total/sec 网卡流量：发送 + 接收，字节

10.% Free Space 逻辑分区可用空间，百分比（物理磁盘 IO 由于 RAID 级别不同，或者有的机器没有 RAID，无法定义统一的监控阈值）

==================================================

CPU:

%Processor Time

%Priviliaged Time

CPU 在特权模式下处理线程所花的时间百分比。一般的系统服务，进城管理，内存管理等一些由操作系统自行启动的进程属于这类

%User Time

与 %Privileged Time 计数器正好相反，指的是在用户状态模式下（即非特权模式）的操作所花的时间百分比。如果该值较大，可以考虑是否通过算法优化等方法降低这个值。如果该服务器是数据库服务器，导致此值较大的原因很可能是数据库的排序或是函数操作消耗了过多的 CPU 时间，此时可以考虑对数据库系统进行优化。

%DPC Time

处理器在网络处理上消耗的时间，该值越低越好。在多处理器系统中，如果这个值大于 50% 并且 %Processor Time 非常高，加入一个网卡可能会提高性能。

Memory:

Available Bytes

另一个监控内存情况的计数器就是这个。这个值最少最少也得大于 5M，因为 sqlserver 需要始终维持 5-10m 的自由内存用于分配，当这个值低于 5m 的时候，那 sqlserver 可能会因为缺少内存而产生性能瓶颈。

Pages/sec

该计数器显示由于页面不在物理内存中而需要从磁盘读取的页面数。Pages/sec 的值很大不一定表明内存有问题，而可能是运行使用内存映射文件的程序所致，操作系统经常会利用磁盘交换的方式提高系统可用的内存量或是提高内存的使用效率。（注意该计数器与 Page Faults/sec 的区别，后者只表明数据不能在内存的指定工作集中立即使用，包括硬错误和软错误）

Page Faults/sec 计数器可以确保磁盘活动不是由分页导致的。在 Windows 中，换页的原因包括：配置进程占用了过多内存 或者 文件系统活动。

如果在同一硬盘上有多个逻辑分区，需要使用 Logical Disk 计数器而非 Physical Disk 计数器。查看逻辑磁盘计数器有助于确定哪些文件被频繁访问。当发现磁盘有大量读 / 写活动时，请查看读写专用计数器以确定导致每个逻辑卷负荷增加的磁盘活动类型，例如，Logical Disk: Disk Write Bytes/sec。

Page Input/sec

表示为了解决硬错误而写入硬盘的页数（参考值：>=Page Reads/sec）

Page Reads/sec

表示为了解决硬错误而从硬盘上读取的页数。（参考值： <=5）

如果怀疑有内存泄露，请监视 Memory/Available Bytes 和 Memory/ Committed Bytes，以观察内存行为，并监视你认为可能在泄露内存的进程的 Process/ Private Bytes、Process/ Working Set 和 Process/ Handle Count。如果怀疑是内核模式进程导致了泄露，则还应该监视 Memory/ Pool Nonpaged Bytes、Memory/ Pool Nonpaged Allocs 和 Process(process_name)/ Pool Nonpaged Bytes

如果发生了内存泄漏, process\private bytes 计数器和 process\working set 计数器的值往往会升高, 同时 avaiable bytes 的值会降低

private Bytes

是指进程所分配的无法与其他进程共享的当前字节数量。该计数器主要用来判断进程在性能测试过程中有无内存泄漏。

例如：对于一个 IIS 之上的 web 应用，我们可以重点监控 inetinfo 进程的 Private Bytes，如果在性能测试过程中，该进程的 Private Bytes 计数器值不断增加，或是性能测试停止后一段时间，该进程的 Private Bytes 仍然持续在高水平，则说明应用存在内存泄漏。

Disk：

PhysicalDisk\Avg. Disk sec/Read

以秒计算的在此盘上读取数据的所需平均时间。

Physical Disk\ Disk Reads/sec

在读取操作时从磁盘上传送的字节平均数。

PhysicalDisk\ Avg. Disk sec/Write

以秒计算的在此盘上写入数据的所需平均时间。

Physical Disk\ DiskWrites/sec

在写入操作时从磁盘上传送的字节平均数。

Physical Disk\ Avg.Disk sec/Transfer

反映磁盘完成请求所用的时间。较高的值表明磁盘控制器由于失败而不断重试该磁盘。这些故障会增加平均磁盘传送时间。

%Disk Time 和 Avg.Disk Queue Length

RAID 磁盘中的 % Disk Time 计数器会指示大于 100% 的值。如果出现这种情况，则使用 PhysicalDisk: Avg.Disk Queue Length 计数器来确定等待进行磁盘访问的平均系统请求数量。

如果不是 RAID，则使用 % Disk Time 和 Current Disk Queue Length 计数器确定是否磁盘存在瓶颈，如果这两个计数器的值一直很高，则可能是磁盘存在瓶颈

Physical Disk：

DiskTransfers/sec 磁盘 IOPS

% Disk Time 当前物理磁盘利用率，如果是 RAID，该值会大于 100%

Current Disk Queue Length 等待进行磁盘访问的当前系统请求数量

Avg.Disk Queue Length 等待进行磁盘访问的平均系统请求数量，用于 RAID