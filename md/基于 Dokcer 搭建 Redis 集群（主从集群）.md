> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/niceyoo/p/14118146.html)

最近陆陆续续有不少园友加我好友咨询 redis 集群搭建的问题，我觉得一定是之前写的这篇 [《基于 Docker 的 Redis 集群搭建》](https://www.cnblogs.com/niceyoo/p/13011626.html) 文章有问题了，所以我花了几分钟浏览之前的文章总结了下面几个问题：

*   redis 数量太少，只创建了 3 个实例；
*   由于只有 3 个实例，所以全部只能是主节点，无法体现集群主从关系；
*   如何搭建主从集群？如何分配从节点？

基于之前的文章，我想快速的过一下这几个问题，本文基于 Docker + Redis 5.0.5 版本，通过 cluster 方式创建一个 6 个 redis 实例的主从集群，当然文章会指出相应的参数说明，这样即便是创建 9 个实例的集群方式也是一样的。

#### 1、拉取 Redis 镜像

基于 Redis：5.0.5 版本，执行如下指令：

```
docker pull redis:5.0.5
```

#### 2、创建 6 个 Redis 容器

创建 6 个 Redis 容器：

*   redis-node1：6379
*   redis-node2：6380
*   redis-node3：6381
*   redis-node4：6382
*   redis-node5：6383
*   redis-node6：6384

执行命令如下：

```
docker create --name redis-node1 --net host -v /data/redis-data/node1:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-1.conf --port 6379docker create --name redis-node2 --net host -v /data/redis-data/node2:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-2.conf --port 6380docker create --name redis-node3 --net host -v /data/redis-data/node3:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-3.conf --port 6381docker create --name redis-node4 --net host -v /data/redis-data/node4:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-4.conf --port 6382docker create --name redis-node5 --net host -v /data/redis-data/node5:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-5.conf --port 6383docker create --name redis-node6 --net host -v /data/redis-data/node6:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-6.conf --port 6384
```

部分参数解释：

*   --cluster-enabled：是否启动集群，选值：yes、no
*   --cluster-config-file 配置文件. conf ：指定节点信息，自动生成
*   --cluster-node-timeout 毫秒值： 配置节点连接超时时间
*   --appendonly：是否开启持久化，选值：yes、no

执行命令截图：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210222843978.png)

#### 3、启动 Redis 容器

执行命令如下：

```
docker start redis-node1 redis-node2 redis-node3 redis-node4 redis-node5 redis-node6
```

启动截图如下：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210223420315.png)

#### 4、组建 Redis 集群

进入任意一个 Redis 实例：

```
# 这里以 redis-node1 实例为例docker exec -it redis-node1 /bin/bash
```

执行组件集群的命令：

```
# 组建集群,10.211.55.4为当前物理机的ip地址redis-cli --cluster create 10.211.55.4:6379 10.211.55.4:6380 10.211.55.4:6381 10.211.55.4:6382 10.211.55.4:6383 10.211.55.4:6384 --cluster-replicas 1
```

执行命令截图如下：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210223827327.png)

创建成功后，通过 redis-cli 查看一下集群节点信息：

```
root@CentOS7:/data# redis-cli127.0.0.1:6379> cluster nodes
```

执行命令截图如下：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210224555085.png)

#### 4、关于 Redis 集群搭建

我们再回到创建集群的命令上：

```
redis-cli --cluster create 10.211.55.4:6379~6384 --cluster-replicas 1
```

大家着重看这个参数 **--cluster-replicas 1**，参数后面的数字表示的是主从比例，比如这里的 1 表示的是主从比例是 1:1，什么概念呢？

也就是 1 个主节点对应几个从节点，现有 6 个实例，所以主从分配就是 3 个 master 主节点，3 个 slave 从节点。

> 主节点最少 3 个，3 个才能保证集群的健壮性。

如果 **--cluster-replicas 2** 呢？

那么主从比例就是 1:2，也就是 1 个主节点对于应 2 个从节点。

即：3(master) + 6(slave) = 9 个 Redis 实例。

如果不足 9 个 Redis 实例，但是参数指定为 2 会怎么样？

报错信息如下：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210230413314.png)

提示已经很清楚了，Redis 集群至少需要 3 个主节点。那么从节点就需要有 6 个，所以最后说：至少需要 9 个节点。

好的，至少 3 个主节点的要求我不继续刚了，但是我想 4 个主节点，2 个从节点，这总该可以了吧？

4 个主节点满足你：

```
# 进入一个启动的 reids 实例，这里以 redis-node1 实例为例docker exec -it redis-node1 /bin/bash
```

执行组建集群的命令：

```
redis-cli --cluster create 10.211.55.4:6379 10.211.55.4:6380 10.211.55.4:6381 10.211.55.4:6382  --cluster-replicas 0
```

指定 4 个没有从节点的主节点，这样你就有 4 个主节点了：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210231122697.png)

剩下的两个从节点怎么办呢？手动添加。

怎么添加？手动添加！

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210231505299.png)

看到这些 master 节点的 id 了吗，只需要把 slave 指定给他们就可以了。

继续执行如下命令：

```
redis-cli --cluster add-node 10.211.55.4:6383 10.211.55.4:6379  --cluster-slave --cluster-master-id b0c32b1dae9e7b7f7f4b74354c59bdfcaa46f30aredis-cli --cluster add-node 10.211.55.4:6384 10.211.55.4:6379  --cluster-slave --cluster-master-id 111de8bed5772585cef5280c4b5225ecb15a582e
```

将两个 Redis 实例塞给其他主节点了：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210232253318.png)

最后我们进入 redis-cli，通过 cluster nodes 查看一下节点信息：

![](https://gitee.com/niceyoo/blog/raw/master/img/image-20201210232445938.png)

看到这，你学废了吗？再学不废，下期我可要录视频了。。。

博客园：[https://niceyoo.cnblogs.com/](https://niceyoo.cnblogs.com/)