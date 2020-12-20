> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/keatsCoder/p/14159751.html)

作者：[@后青春期的 Keats](https://www.cnblogs.com/keatsCoder)  
本文为作者原创，转载请注明出处：[https://www.cnblogs.com/keatsCoder/p/14159751.html](https://www.cnblogs.com/keatsCoder/p/14159751.html)

*   [大数据概论](#_caption_0)
    *   [什么是大数据](#_caption0)
    *   [大数据的特点](#_caption1)
*   [大数据应用场景](#_caption_1)
    
*   [大数据发展前景](#_caption_2)
    
*   [简介](#_caption_3)
    *   [三大发行版本](#_caption2)
    *   [优势](#_caption3)
*   [组成](#_caption_4)
    *   [HDFS 架构概述](#_caption4)
    *   [YARN 架构概述](#_caption5)
    *   [MapReduce 架构概述](#_caption6)
*   [大数据技术生态体系](#_caption_5)
    
*   [安装](#_caption_6)
    *   [虚拟机网络配置](#_caption7)
*   [运行模式](#_caption_7)
    *   [本地模式](#_caption8)
    *   [伪分布式模式](#_caption9)
*   [** 完全分布式](#_caption_8)
    *   [克隆虚拟机](#_caption10)
    *   [安装 JDK（scp 命令学习）](#_caption11)
    *   [集群配置](#_caption12)
    *   [集群单点启动](#_caption13)
    *   [群起集群](#_caption14)
    *   [测试集群](#_caption15)
    *   [集群时间同步](#_caption16)

大数据简介，概念部分
==========

概念部分，建议之前没有任何大数据相关知识的朋友阅读

大数据概论
-----

### 什么是大数据

大数据（Big Data）是指**无法在一定时间范围**内用常规软件工具进行捕捉、管理和处理的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化能力的**海量、高增长率和多样化的信息资产**

粗略解读以下

*   常规软件工具：例如 JavaEE、Mysql（500-1000w 数据）即使构建 Mysql 集群，集群中节点的数量也不是无限增加的。
*   海量、高增长率：数据本身基数大，每天新加入的数据也多
*   多样化：除了文本数据外，还有图片、视频等数据

主要解决**海量数据的存储**和**海量数据的分析计算**问题

### 大数据的特点

1.  大量：根据 IDC “数字宇宙” 报告 预计到 2020 年，全球数据使用量将达到 35.2ZB
2.  高速：在海量的数据面前，处理数据的效率就是企业的生命
3.  多样
    *   结构化数据：数据库、文本为主
    *   非结构化数据：网络日志、音频、视频、图片、地理位置信息
4.  低价值密度：价值密度的高低与数据总量的大小成反比，快速对有价值数据 “提纯”

大数据应用场景
-------

1.  物流仓储：大数据分析系统助力商家精细化运营、提升销量、节约成本
2.  零售：分析用户消费习惯，为用户购买商品提供方便，从而提升商品销量
    *   国外案例：纸尿裤 + 啤酒，分析过往的商超订单发现纸尿裤 + 啤酒同时出现的概率很高，因此将这两样物品放在一起，刺激更多的人消费
3.  旅游：深度结合大数据能力与旅游行业需求，共建旅游产业智慧管理、智慧服务和智慧营销的未来
4.  商品广告推荐：给用户推荐可能喜欢的商品（千人千面）
5.  保险、金融、房地产：数据挖掘、风险预测

大数据发展前景
-------

十八大：实施国家大数据发展战略

十九大：推动互联网、大数据、人工智能和实体经济深度融合

Hadoop
======

简介
--

Hadoop 是 Apache 基金会开发的分布式系统基础架构，主要解决海量数据的存储和分析计算问题。与大数据所研究的方向一样。广义上来说，Hadoop 通常指的是 Hadoop 的生态圈。我理解就和 Javaer 常说的 Spring Cloud 一样，并不特指一个技术，而是一些携手解决一个复杂问题的集合

创始人 Doug Cutting ，借鉴谷歌三篇论文

GFS ---> HDFS 解决了数据存储的问题

Map-Reduce ---> MR 解决了数据分析计算的问题

BigTable ---> HBase NoSQL 数据库

### 三大发行版本

Apache

最原始，最基础版本，适合入门学习

Cloudera

CDH 版本、2008 年成立，2009 年 Doug Cutting 加盟。 主要在大型互联网企业中使用，免费使用，付费维护

Hortonworks

2011 年成立，文档较好，市场份额小

### 优势

*   高可靠性：底层维护多个数据副本（default 3）
*   高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点
*   高效性：在 MapReduce 的思想下，Hadoop 是并行工作的
*   高容错性：能够自动将失败的任务重新分配

组成
--

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162808048-1201253601.png)

2.x 时代的 Hadoop 将数据的计算分析和资源调度进行解耦，独立出来 Yarn 专门负责 CPU、内存、硬盘、网络等资源的调度

### HDFS 架构概述

*   NameNode(nn) 存储文件的元数据，如文件名，目录结构，文件属性（生成时间，副本数，文件权限）以及每个文件的块列表和块所在的 DataNode 等
*   DataNode(dn) 在本地文件系统存储文件块数据，以及块数据的校验和
*   Secondary NameNode(2nn) 监控 HDFS 状态的辅助后台程序，每隔一段时间获取 HDFS 元数据的快照

### YARN 架构概述

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162807507-24711042.png)

### MapReduce 架构概述

MapReduce 将计算过程分为两个阶段：Map 和 Reduce

1.  Map 阶段并行处理数据
2.  Reduce 阶段对 Map 结果进行汇总

大数据技术生态体系
---------

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162806721-1811097948.png)

安装
--

**建议在新的虚拟机完成，方便后续克隆、搭建集群**

**安装 hadoop 之前建议设置 linux IP 为静态 IP、必须安装 Java 以及配置环境变量、建议关闭防火墙（自己测试的时候）**

### 虚拟机网络配置

我自己的网络配置 `vim /etc/sysconfig/network-scripts/ifcfg-ens33`， 网络采用 NAT 方式。一般 NAT 模式对应的虚拟网卡是 vmnet8，

主机中 vmnet8 对应的虚拟网卡 ipv4 地址需要设置静态 IP， 并且该 IP 与虚拟机网络编辑器中 vmnet8 的 子网 IP 、 以及下面配置中 IPADDR，GATEWAY，DNS1 前三段要一致。且第四段不能一样。如图所示

1.  主机虚拟网卡配置

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162806022-1009058225.png)

2.  虚拟机网络编辑器

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162805731-1331339261.png)

3.  虚拟机网络配置文件

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"

UUID="5f66ee29-f43f-4761-abec-bd0656e25e09"
DEVICE="ens33"
ONBOOT="yes"
IPV6_PRIVACY="no"

IPADDR="192.168.100.104"
GATEWAY="192.168.100.2"
DNS1="192.168.100.2"


```

我安装的版本是 [2.10.1](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz)

下载上面的压缩包在 linux 服务器，解压后放在 /opt/module 目录下

配置环境变量

```
export HADOOP_HOME=/opt/module/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin


```

刷新 profile 文件

```
source /etc/profile


```

测试安装结果

```
# hadoop
Usage: hadoop [--config confdir] [COMMAND | CLASSNAME]
  CLASSNAME            run the class named CLASSNAME
 or
  where COMMAND is one of:
  fs                   run a generic filesystem user client
  version              print the version
  jar <jar>            run a jar file
                       note: please use "yarn jar" to launch
                             YARN applications, not this command.
  checknative [-a|-h]  check native hadoop and compression libraries availability
  distcp <srcurl> <desturl> copy file or directories recursively
  archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
  classpath            prints the class path needed to get the
                       Hadoop jar and the required libraries
  credential           interact with credential providers
  daemonlog            get/set the log level for each daemon
  trace                view and modify Hadoop tracing settings

Most commands print help when invoked w/o parameters.


```

运行模式
----

*   本地模式，单节点 Java 进程，一般用于调试
    
*   伪分布式模式，适合计算机性能不是非常强劲的朋友使用（16GB 内存以下）
    
*   分布式
    

### 本地模式

如果你的 Hadoop 包是从官方下载的正式包，默认情况下。Hadoop 配置的都是本地运行模式

#### 官方 Grep 案例

[http://hadoop.apache.org/docs/r2.10.1/hadoop-project-dist/hadoop-common/SingleCluster.html](http://hadoop.apache.org/docs/r2.10.1/hadoop-project-dist/hadoop-common/SingleCluster.html)

```
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar grep input output 'dfs[a-z.]+'
$ cat output/*


```

出现下面的内容，则本地模式运行成功

```
1       dfsadmin


```

正如我们看到的那样

#### 官方 WordCount 案例

1.  创建输入目录（源目录） wcinput , 新建文本文件 wc.txt
    
    ```
    # mkdir wcinput
    # cd wcinput/
    # touch wc.input
    # vim wc.input
    
    lvbanqihao ake libai libai
    hanxin wuya hanxin zhangsan
    direnjie guanyu guanyu zhangfei
    chengjisihan jing
    
    
    ```
    
2.  执行 examples 中的 wordcount 功能
    
    ```
    # hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar wordcount wcinput/ wcoutput
    
    
    ```
    
3.  查看结果
    
    ```
    # cat wcoutput/*
    
    ake     1
    chengjisihan    1
    direnjie        1
    guanyu  2
    hanxin  2
    jing    1
    libai   2
    lvbanqihao      1
    wuya    1
    zhangfei        1
    zhangsan        1
    
    
    ```
    

### 伪分布式模式

修改 ${HADOOP_HOME}/etc/hadoop 下的配置文件。含义见注释，

#### HDFS 的配置和操作

##### 配置

###### core-site.xml

```
<!-- 指定HDFS中NameNode的地址 -->
<property>
<name>fs.defaultFS</name>
    <value>hdfs://linux101:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/module/hadoop/data/tmp</value>
</property>


```

###### hdfs-site.xml

```
<!-- 指定HDFS副本的数量 -->
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>


```

###### hadoop-env.sh

修改 JAVA_HOME 路径为 JDK 路径

1.  通过 `echo $JAVA_HOME` 命令显示已配置的 JAVA_HOME 路径。复制
2.  vim 打开 hadoop-env.sh 修改

##### 启动集群

*   格式化 NameNode

仅在第一次启动需要 (格式化 NameNode，会产生新的集群 id, 导致 NameNode 和 DataNode 的集群 id 不一致，集群找不到已往数据。所以，格式 NameNode 时，一定要先删除 data 数据和 log 日志，然后再格式化 NameNode)

```
bin/hdfs namenode -format


```

*   启动 NameNode

```
sbin/hadoop-daemon.sh start namenode


```

*   启动 DataNode

```
sbin/hadoop-daemon.sh start datanode


```

##### 查看集群

1.  首先通过 JDK 的 jps 命令查看 NameNode 和 DataNode 进程是否启动
2.  接着访问该服务器的 50070 端口（确保防火墙已经关闭，并且主机和虚拟机可以互相访问）
3.  查看日志文件 ${HADOOP_HOME}/logs

##### 操作集群

在 HDFS 文件系统上**创建**一个 input 文件夹

```
bin/hdfs dfs -mkdir -p /user/keats/input


```

将测试文件内容**上传**到文件系统上

```
bin/hdfs dfs -put wcinput/wc.input /user/keats/input/


```

**查看**上传的文件是否正确

```
bin/hdfs dfs -ls  /user/keats/input/
bin/hdfs dfs -cat  /user/keats/ input/wc.input


```

运行 MapReduce 程序

```
bin/hadoop jar
share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/keats/input/ /user/keats/output


```

查看输出结果

```
bin/hdfs dfs -cat /user/keats/output/*


```

将测试文件内容**下载**到本地

```
hdfs dfs -get /user/keats/output/part-r-00000 ./wcoutput/


```

**删除**输出结果

```
hdfs dfs -rm -r /user/keats/output


```

#### YARN 的操作和配置

##### 配置

配置 JAVA_HOME

${HADOOP_HOME}/etc/hadoop/

*   yarn-env.sh
*   mapred-env.sh

配置 yarn-site.xml

```
<!-- Reducer获取数据的方式 -->
<property>
 		<name>yarn.nodemanager.aux-services</name>
 		<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
<name>yarn.resourcemanager.hostname</name>
<value>linux101</value>
</property>


```

配置： (对 mapred-site.xml.template 重新命名为) mapred-site.xml

```
mv mapred-site.xml.template mapred-site.xml

<!-- 指定MR运行在YARN上 -->
<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
</property>


```

##### 启动集群

*   启动前必须保证 NameNode 和 DataNode 已经启动
    
*   启动 ResourceManager
    
    ```
    sbin/yarn-daemon.sh start resourcemanager
    
    
    ```
    
*   启动 NodeManager
    
    ```
    sbin/yarn-daemon.sh start nodemanager
    
    
    ```
    

#### 配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下

1.  配置 mapred-site.xml

```
<!-- 历史服务器端地址 -->
<property>
<name>mapreduce.jobhistory.address</name>
<value>linux101:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
  <name>mapreduce.jobhistory.webapp.address</name>
  <value>linux101:19888</value>
</property>


```

2.  启动历史服务器

```
sbin/mr-jobhistory-daemon.sh start historyserver


```

3.  jps 查看历史服务器是否启动
    
4.  查看 JobHistory [http://linux101:19888/jobhistory](http://linux101:19888/jobhistory)
    

#### 配置日志的聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到 HDFS 系统上。

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

**注意**：开启日志聚集功能，需要重新启动 NodeManager 、ResourceManager 和 HistoryManager

##### 配置 yarn-site.xml

```
<!-- 日志聚集功能使能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>


```

##### 重启 NodeManager 、ResourceManager 和 HistoryManager

没有 restart 命令，只能先关后开 或者自己写个脚本

关闭

```
sbin/yarn-daemon.sh stop resourcemanager
sbin/yarn-daemon.sh stop nodemanager
sbin/mr-jobhistory-daemon.sh stop historyserver


```

关闭完成后，执行 jps 进行验证

打开

```
sbin/yarn-daemon.sh start resourcemanager
sbin/yarn-daemon.sh start nodemanager
sbin/mr-jobhistory-daemon.sh start historyserver


```

##### 测试

删除 HDFS 上已经存在的输出文件

```
bin/hdfs dfs -rm -R /user/keats/output


```

执行 wordcount

```
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar wordcount /user/keats/input /user/keats/output


```

查看日志 [http://linux101:19888/jobhistory](http://linux101:19888/jobhistory)

#### 配置文件说明

Hadoop 配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值

##### 默认配置文件

<table><thead><tr><th>要获取的默认文件</th><th>文件存放在 Hadoop 的 jar 包中的位置</th></tr></thead><tbody><tr><td>[core-default.xml]</td><td>hadoop-common-2.7.2.jar/ core-default.xml</td></tr><tr><td>[hdfs-default.xml]</td><td>hadoop-hdfs-2.7.2.jar/ hdfs-default.xml</td></tr><tr><td>[yarn-default.xml]</td><td>hadoop-yarn-common-2.7.2.jar/ yarn-default.xml</td></tr><tr><td>[mapred-default.xml]</td><td>hadoop-mapreduce-client-core-2.7.2.jar/ mapred-default.xml</td></tr></tbody></table>

##### 自定义配置文件

core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml 四个配置文件存放在 $HADOOP_HOME/etc/hadoop 这个路径上，用户可以根据项目需求重新进行修改配置

** 完全分布式
--------

### 克隆虚拟机

一般都通过克隆虚拟机的方式来模拟多台物理机，去模拟完全分布式。（钱多任性的土豪除外）

*   首先确保被克隆虚拟机以及配置静态 IP、安装 JDK、关闭防火墙、配置好了 hosts 文件
    
*   需要克隆 3 台虚拟机
    
*   ip 对应着 102、103、104
    
*   更改主机名， centos7 使用这个命令： `hostnamectl set-hostname linux103`
    
*   另外 vmware15 + centos7 在修改 IP 并重启后， mac 地址会自动变化，这个不用手动修改（视频中 centos6 老师是手动改的）
    

全部搞定后，主机通过 moba 是可以通过虚拟机主机名连接任何一个虚拟机的（主机的 hosts 文件也需要配置）

### 安装 JDK（scp 命令学习）

#### scp（secure copy）安全拷贝

这块其实按照我刚才的克隆操作，jdk 已经安装配置好。但是视频中老师是克隆的空虚拟机。我想大概是主要为了教大家 scp (secure copy) 命令

```
scp -r [username@hostname1:]/x/xxx [username@hostname2:]/x/xxx


```

表示安全的从 hostname1 递归拷贝文件到 hostname2 服务器。

其中 [username@hostname1:] 表示可以省略，省略的时候表示当前服务器本地的文件夹 / 文件。username 表示远程主机的用户名，hostname 表示远程主机的主机名

#### rsync 远程同步工具

与 scp 不同的地方有两处

一是该工具仅同步差异的文件。相同的文件不做操作

二是该工具只能操作本机和另外一台机器之间的同步，不能操作两个其他服务器

**注意**：如果本机文件路径对应的是一个文件，而外部机器对应的是一个不存在的文件夹。则该文件内容会被拷贝成文件夹名称的文件

```
rsync -rvl /x/xxx [username@hostname2:]/x/xxx


```

<table><thead><tr><th>选项</th><th>功能</th></tr></thead><tbody><tr><td>-r</td><td>递归</td></tr><tr><td>-v</td><td>显示复制过程</td></tr><tr><td>-l</td><td>拷贝符号连接</td></tr></tbody></table>

#### xsync 集群分发脚本

学会了 rsync 命令之后，就可以进行集群中两个服务器之间的文件同步了，但是对于正式环境中动辄几十成百上千台服务器来说，手敲命令同步文件肯定不现实。因此需要写一个分发命令的脚本

该脚本会读取输入脚本后的第一个参数（要分发的文件所在目录）

在当前用户目录下，如果是 root 则在 root 目录下创建 bin 文件夹，然后创建 xsync 文件，内容如下

```
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for((host=103; host<105; host++)); do
        echo ------------------- linux$host --------------
        rsync -rvl $pdir/$fname $user@linux$host:$pdir
done


```

修改脚本具有可执行权限 `chmod +x xsync`

### 集群配置

#### 集群部署规划

1.  NameNode 和 SecondaryNameNode 占用的内存是相当的。比较耗内存。因此需要分开
2.  ResourceManager 也比较耗内存

<table><thead><tr><th></th><th>linux102</th><th>linux103</th><th>linux104</th></tr></thead><tbody><tr><td>HDFS</td><td>NameNode DataNode</td><td>DataNode</td><td>SecondaryNameNode DataNode</td></tr><tr><td>YARN</td><td>NodeManager</td><td>ResourceManager NodeManager</td><td>NodeManager</td></tr></tbody></table>

#### 配置集群

##### 核心配置文件 core-site.xml

```
<!-- 指定HDFS中NameNode的地址 -->
<property>
		<name>fs.defaultFS</name>
      <value>hdfs://linux102:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop/data/tmp</value>
</property>


```

HDFS 配置文件

##### 配置 hadoop-env.sh

```
vi hadoop-env.sh
export JAVA_HOME=/opt/module/jdk1.8


```

##### 配置 hdfs-site.xml

```
<property>
		<name>dfs.replication</name>
		<value>3</value>
</property>

<!-- 指定Hadoop辅助名称节点主机配置 -->
<property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>linux104:50090</value>
</property>


```

YARN 配置文件

##### 配置 yarn-env.sh

```
vi yarn-env.sh
export JAVA_HOME=/opt/module/jdk1.8


```

##### 配置 yarn-site.xml

```
<!-- Reducer获取数据的方式 -->
<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>linux103</value>
</property>


```

MapReduce 配置文件

##### 配置 mapred-env.sh

```
vi mapred-env.sh

export JAVA_HOME=/opt/module/jdk1.8


```

##### 配置 mapred-site.xml

在该文件中增加如下配置

```
<!-- 指定MR运行在Yarn上 -->

<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>


```

##### 在集群上分发配置好的 Hadoop 配置文件

xsync /opt/module/hadoop/

##### 查看文件分发情况

cat /opt/module/hadoop/etc/hadoop/core-site.xml

### 集群单点启动

如果集群是第一次启动，需要**格式化 NameNode** . 如果格式化遇到问题，需要重新排查问题后，重新格式化

因为我的虚拟机在复制之前，做过伪分布式的测试。残留了 data logs 目录，需要**删除三个虚拟机上的 data/ logs/ 目录**，如果你是按照我的步骤走的，也需要统统删除后，再执行格式化操作

```
hadoop namenode -format


```

之后启动三个节点的 102 节点的 namenode 和 三个节点的 datanode， 打开 linux102:50070 查看是否启动成功

#### 配置 ssh 登录

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162804969-1772100510.png)

生成公钥和私钥，进入当前用户根目录下的 .ssh 目录，我使用的 root 用户

```
ssh-keygen -t rsa


```

将公钥拷贝到要免密登录的目标机器上

```
ssh-copy-id linux102
ssh-copy-id linux103
ssh-copy-id linux104


```

对于当前机器（例如 Linux102）通过 ssh 访问自己时，也是需要密码的。想要无秘访问也需要将自己的公钥追加到自己的认证文件后面

ssh 文件夹下（~/.ssh）的文件功能解释

<table><thead><tr><th>known_hosts</th><th>记录 ssh 访问过计算机的公钥 (public key)</th></tr></thead><tbody><tr><td>id_rsa</td><td>生成的私钥</td></tr><tr><td>id_rsa.pub</td><td>生成的公钥</td></tr><tr><td>authorized_keys</td><td>存放授权过得无密登录服务器公钥</td></tr></tbody></table>

### 群起集群

配置 slaves

```
cd /opt/module/hadoop/etc/hadoop/
vim slaves


```

配置所有 datanode 服务器，不允许有空格

```
linux102
linux103
linux104


```

分发给 103 102

```
xsync slaves


```

**启动集群**

```
sbin/start-dfs.sh


```

关闭集群

```
sbin/stop-dfs.sh


```

**启动 YARN**

老师视频里提到必须使用 103 节点起，我实测 2.10.1 版本是可以在 102 服务器起的，不知是 sh 文件更新还是 root 账户的原因

```
sbin/start-yarn.sh


```

Web 端测试 SecondaryNameNode

浏览器中输入：[http://linux104:50090/status.html](http://linux104:50090/status.html)

### 测试集群

上传小文件

```
hdfs dfs -put wcinput/wc.input /


```

上传大文件

```
hdfs dfs -put /opt/software/hadoop-2.10.1.tar.gz /


```

大文件占了多个 block，如果下载的话多个块又会被合并下载下来

![](https://img2020.cnblogs.com/blog/1654189/202012/1654189-20201219162803783-1316639866.png)

实际文件存储在 ${HADOOP_HOME}/data/tmp 路径下的子子子子子子目录中，感兴趣的读者可以进去看看

```
/opt/module/hadoop/data/tmp/dfs/data/current/BP-1473062949-192.168.100.102-1608214270058/current/finalized/subdir0/subdir0


```

### 集群时间同步

#### 时间服务器配置

1.  检查 ntp 是否安装 `rpm -qa|grep ntp` 机器都是克隆的，查下 102 就可
    
    ```
    ntp-4.2.6p5-10.el6.centos.x86_64
    fontpackages-filesystem-1.41-1.1.el6.noarch
    ntpdate-4.2.6p5-10.el6.centos.x86_64
    
    
    ```
    
2.  修改 ntp 配置文件
    
    1.  授权局域网网段上的所有机器可以从这台机器上查询和同步时间 `vim /etc/ntp.conf`
        
    2.  当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步
        
        ```
        # 打开这段注释，将 IP 第三段改成自己虚拟机局域网第三段
        restrict 192.168.100.0 mask 255.255.255.0 nomodify notrap
        
        # 添加下面的内容：当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步
        server 127.127.1.0
        fudge 127.127.1.0 stratum 10
        
        
        ```
        
3.  修改 /etc/sysconfig/ntpd 文件，让硬件时间与系统时间一起同步
    
    ```
    vim /etc/sysconfig/ntpd
    
    增加内容
    SYNC_HWCLOCK=yes
    
    
    ```
    
4.  重启 ntpd 服务 `service ntpd status` 查看状态 start 启动
    
5.  设置 ntpd 开机自启 `chkconfig ntpd on`
    

#### 其他服务器配置

1 天 1 次

```
0 0 * * 1-7 /usr/sbin/ntpdate linux102


```

修改服务器时间

```
date -s "2017-9-11 11:11:11"


```

经过 10 分钟等待，看时间是否同步 (建议测试修改 cron 表达式，使同步间隔缩小)

```
date


```