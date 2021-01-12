> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/camelliabloglyf/p/14262631.html)

　　上个月，有个朋友问我说 Sql Sever 向 Mysql 迁移有什么好的经验分享，他们公司客户明确提出不再提供 Windows 服务器，现在计划 Mysql 迁移。我说 Mysql 迁移成本太高了，不妨可以了解一下 SQL Server On Linux 再做决定。于是，我把之前给运维分享的 Word 文档发给了他，告诉他，如果可以接受一些不支持的功能，选择成本，风险小的，如果项目中用到的技术知识刚好避开了那些不支持的功能，3~5 个小时可以完成一个项目的迁移。我们公司也有案例，在 Linux 平台上，同时安装了 Sqlserver2017 和 Mysql, 旧功能升级 Sql Server，新功能用 Mysql。

　　上周他很高兴的告诉我，他们公司最终选择了 SQL Server On Linux，已经完成了一个大项目的升级了，目前使用稳定，项目在正常运行中, 他说他今年升职加薪有戏了。后来了解到，他们选择 Mysql 迁移，是因为他们不知道 Sqlserver2017 及以上版本也是支持 Linxu 平台的，于是强烈建议我把内容分享到博客园，让一些人少走一些弯路。

背景
==

　　在过去的 20 多年中，微软的各大产品靠 Windows 绑定市场，众多的微软 ISV 围绕着 Windows 开发系列产品，形成一个以 Windows 为核心的生态系统。随着互联网的发展，出现了 Google,Facebook,Tencent,Baidu,Alibaba 都是以 Linux 操作系统构建的产品生态系统，他们不再是具体的产品，而是提供服务，而且服务所用技术都是开源的，和原来 Windows 的生态不是同一个维度的世界，微软封闭的生态系统只有慢慢的瓦解。微软也意识到问题的严重性，换了那个称 Linux 为毒瘤的 CEO **史蒂夫 · 鲍尔默**，用领导微软云的**萨提亚 · 纳德拉**来带领微软走出原来封闭的生态系统，走入开放的云生态系统。

　　a)     云计算机会比 Windows 大，Windows 占微软的营收越来越少。

　　b)     服务器版操作系统市场份额基本是 Linux 稳占第一把交椅，微软要让自家的数据库市场份额扩大来挤占其他数据库的份额，必然要做出 SQL Server on Linux 的决定。

支持的平台
=====

　　SQL Server 在 Red Hat Enterprise Linux (RHEL)、SUSE Linux Enterprise Server (SLES) 和 Ubuntu 上受支持。 此外，它也可作为 Docker 映像提供，可在 Linux 上的 Docker 引擎或用于 Windows/Mac 的 Docker 上运行。

| 

**平台**

 | 

**支持的版本**

 |
| 

**Red Hat Enterprise Linux**

 | 

7.3、7.4、7.5、7.6、8

 |
| 

**SUSE Linux Enterprise Server**

 | 

v12 SP2

 |
| 

**Ubuntu**

 | 

**16.04****、**18.04

 |
| 

**Docker** **引擎**

 | 

1.8+

 |

 **You need to have at least Ubuntu 16.04 or you will face unmet dependencies problems.**

系统要求
====

　　SQL Server 对 Linux 具有以下系统要求：

| 

内存：

 | 

2 GB

 |
| 

文件系统：

 | 

XFS 或 EXT4（其他文件系统均不受支持，如 BTRFS）

 |
| 

磁盘空间：

 | 

6 GB

 |
| 

处理器速度：

 | 

2 GHz

 |
| 

处理器核心数：

 | 

2 个核心

 |
| 

处理器类型：

 | 

仅兼容 x64

 |

版本选择
====

| 

**SQL Server** **版本**

 | 

**描述**

 |
| 

企业

 | 

SQL Server Enterprise Edition 是高级产品，可提供全面的高端数据中心功能，并具有超快的性能，可为任务关键型工作负载提供高服务水平。可用性组支持总副本（一个主副本，八个辅助副本）

 |
| 

标准

 | 

SQL Server Standard Edition 为部门和小型组织提供了运行其应用程序的基本数据管理，并支持用于内部部署和云的通用开发工具 - 以最少的 IT 资源实现有效的数据库管理。

 |
| 

网页

 | 

SQL Server Web 版是 Web 托管者和 Web VAP 的总拥有成本低的选项，可为小型到大型 Web 属性提供可伸缩性，可负担性和可管理性。

 |
| 

**开发者**

 | 

**SQL Server Developer** **版本使开发人员可以在 SQL Server 之上构建任何类型的应用程序。它包含企业版的所有功能，但已获许可用作开发和测试系统，而不用作生产服务器。SQL Server Developer 是构建和测试应用程序的人们的理想选择。**

 |
| 

速成版

 | 

Express Edition 是入门级的免费数据库，非常适合学习和构建台式机和小型服务器数据驱动的应用程序。对于构建客户端应用程序的独立软件供应商，开发人员和爱好者来说，这是最佳选择。如果需要更高级的数据库功能，则可以将 SQL Server Express 无缝升级到 SQL Server 的其他更高端版本。

 |

```
Choose an edition of SQL Server:
   1. Evaluation (free, no production use rights, 180-day limit)
   2. Developer (free, no production use rights)
   3. Express (free)
   4. Web (PAID)
   5. Standard (PAID)
   6. Enterprise (PAID)
   7. Enterprise Core (PAID)
   8. I bought a license through a retail sales channel and have a product key to enter.
```

脱机安装（推荐使用）
==========

　　下面安装以 **Red Hat** **为例。**

Wget 安装
-------

```
yum -y install wget
```

　　已经安装了就跳过此步。

安装 mssql-server
---------------

　　如果 Linux 计算机无法访问联机存储库，则可以直接下载包文件。 这些包位于 Microsoft 存储库中，地址为 [https://packages.microsoft.com](https://packages.microsoft.com/)

　　a)   创建目录下载 RPM 包

```
mkdir -p /opt/sqlserver2017 
cd /opt/sqlserver2017/
wget https://packages.microsoft.com/rhel/7/mssql-server-2017/mssql-server-14.0.3048.4-1.x86_64.rpm
```

       b)   Yum 安装 mssql-server

```
yum localinstall mssql-server-14.0.3048.4-1.x86_64.rpm
```

　　![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111172935491-1718203676.png)

 配置
---

```
/opt/mssql/bin/mssql-conf setup
```

　　运行 mssql-conf setup，按照提示设置 SA 密码并选择版本。

*   请确保为 SA 帐户指定强密码（最少 8 个字符，包括大写和小写字母、十进制数字和 / 或非字母数字符号）。
*   Developer (free, no production use rights)（版本选择 2，Developer）

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111173308331-1624613040.png)

 验证服务
-----

```
systemctl status mssql-server
```

　　![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111173351525-1833919620.png)

服务启停
----

```
systemctl stop mssql-server
systemctl start mssql-server
systemctl restart mssql-server
```

安装 sqlcmd 和 bcp SQL Server 命令行工具
--------------------------------

 **a)** **下载**

```
wget https://packages.microsoft.com/rhel/7.3/prod/msodbcsql-13.1.6.0-1.x86_64.rpm
wget https://packages.microsoft.com/rhel/7.3/prod/mssql-tools-14.0.5.0-1.x86_64.rpm
```

 **b)** **安装**

```
yum localinstall msodbcsql-13.1.6.0-1.x86_64.rpm
yum localinstall mssql-tools-14.0.5.0-1.x86_64.rpm
```

添加环境变量
------

　　为方便起见，向 PATH 环境变量添加 `/opt/mssql-tools/bin/`。 这样可以在不指定完整路径的情况下运行这些工具。 

　　运行以下命令以修改登录会话和交互式 / 非登录会话的路径：

```
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
```

设置远程连接，打开端口
-----------

　　默认的 SQL Server 端口为 TCP 1433。 如果为防火墙使用的是 **FirewallD**，则可以使用以下命令：

```
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload
```

　　使用 SQL Server 名称 (-S)，用户名 (-U) 和密码 (-P) 的参数运行 sqlcmd。 用户名为 `SA`，密码是在安装过程中为 SA 帐户提供的密码。

```
sqlcmd -S localhost -U SA -P '<YourPassword>'
　sqlcmd -S 192.168.1.XXX -U userName
```

　　接着输入密码：

```
SELECT Name FROM Master..SysDatabases ORDER BY Name
SELECT Name FROM Sys.Databases ORDER BY Name
go
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111174153367-653211830.png)

　　此时可以用 Navicat 或者 SqlServer2017 验证连接情况。

安装设置 Agent（SQL Server 代理）
-------------------------

　　SQL Server Agent 也叫 SQL Server 代理，以前称为 SQL 执行者，这是 SQL Server 的任务日程表。

　　这种服务主要是用于在设定的时间备份、复制数据，以及在自动执行调度表上设置的其他项目。启动这个服务后，设定好在什么时候做什么事，这个服务会让它自动运行，不需要人工干预。

 **a)** **下载**

```
wget https://packages.microsoft.com/rhel/7/mssql-server-2017/mssql-server-agent-14.0.3015.40-1.x86_64.rpm
```

 **b)** **安装**

```
yum localinstall mssql-server-agent-14.0.3015.40-1.x86_64.rpm
```

 **c)** **启用代理（作业）**

　　使用 sqlagent.enabled 设置可启用 SQL Server 代理。 默认情况下，SQL Server 代理处于禁用状态。 如果 mssql.conf 设置文件中不存在 sqlagent.enabled，则 SQL Server 在内部假定已禁用 SQL Server 代理。若要更改此设置，请使用以下步骤：

```
sudo /opt/mssql/bin/mssql-conf set sqlagent.enabled true
sudo systemctl restart mssql-server
```

 **d)** **代理错误日志设置**

　　sqlpagent.errorlogfile 和 sqlpagent.errorlogginglevel 设置允许你分别设置 SQL 代理日志文件路径和日志记录级别。

　　sudo /opt/mssql/bin/mssql-conf set sqlagent.errorfile **<path>**

　　SQL 代理日志记录级别是位掩码值，等于：

　　1 = 错误

　　2 = 警告

　　4 = 信息

　　如果要捕获所有级别，请使用 `7` 作为值。

```
sudo /opt/mssql/bin/mssql-conf set sqlagent.errorlogginglevel 7
```

设置默认语言与排序规则
-----------

　　a)   若要获取支持的排序规则的列表，请运行 sys.fn_helpcollations 函数

```
SELECT NAME FROM SYS.FN_HELPCOLLATIONS()
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111181214090-42922947.png)

　　b)   如果安装时没有指定环境变量参数，会按默认设置安装，字符集会是拉丁字符集，默认语言是英语，此时中国用户需要进行另外设置。

```
systemctl stop mssql-server
/opt/mssql/bin/mssql-conf set-collation
Chinese_PRC_CI_AS
```

　　c)   可以通过预先设置环境变量来按照指定的字符集及本地语言等设置参数，这样的话无需安装后再进行配置。

```
MSSQL_LCID='2052' MSSQL_COLLATION='Chinese_PRC_CI_AS'
/opt/mssql/bin/mssql-conf setup
```

　　d)   查询当前排序规则

```
select serverproperty('Collation')
```

设置内存限制
------

　　使用 memory.memorylimitmb 设置可控制 SQL Server 可用的物理内存量（以 MB 为单位）。 默认值为物理内存的 80%。（我们根据情况而定，更改此设置时，不要将此值设置得太高。 如果不留出足够的内存，则可能会遇到 Linux 操作系统和其他 Linux 应用程序的问题）

　　a)   使用 memory.memorylimitmb 的 set 命令以根用户身份运行 mssql-conf 脚本。 以下示例将 SQL Server 可用的内存更改为 3.25 GB (3328 MB)。

```
sudo /opt/mssql/bin/mssql-conf set memory.memorylimitmb 6656
```

　　b)   重启 SQL Server 服务以应用更改

```
sudo systemctl restart mssql-server
```

更改 TCP 端口
---------

　　使用 network.tcpport 设置可更改 SQL Server 侦听连接的 TCP 端口。 默认情况下，此端口设置为 1433。 若要更改端口，请运行以下命令：

　　a)  使用 “network.tcpport” 的 “set” 命令以根用户身份运行 mssql-conf 脚本

```
sudo /opt/mssql/bin/mssql-conf set network.tcpport <new_tcp_port>
```

　　b)  重启 SQL Server 服务

```
sudo systemctl restart mssql-server
```

　　c)  连接到 SQL Server 后，必须在主机名或 IP 地址后用逗号 (,) 指定自定义端口。 

　　例如，要使用 SQLCMD 进行连接：

　　sqlcmd -S localhost,<new_tcp_port> -U test -P test

```
# iptables -A INPUT -p tcp --dport 1433 -j ACCEPT
# iptables-save > /etc/sysconfig/iptables
# firewall-cmd --add-port=1433/tcp --permanent
# firewall-cmd --reload
```

更改默认数据或日志目录位置
-------------

　　使用 filelocation.defaultdatadir 和 filelocation.defaultlogdir 设置可更改创建新数据库和日志文件的位置。 默认情况下，此位置为 /var/opt/mssql/data。 若要更改这些设置，请使用以下步骤：

　　a)   为新的数据库数据和日志文件创建目标目录。 以下示例创建一个新的 **/mssql/data** 目录

```
mkdir -p /mssql/data
```

　　b)   将目录的所有者和组更改为 **mssql** 用户

 **数据目录的上一级目录必须设置 mssql 用户才会有权限！！！！（与 Mysql 的不同）**

```
sudo chown mssql /mssql
sudo chgrp mssql /mssql
sudo chown mssql /mssql/data
sudo chgrp mssql /mssql/data
```

　　c)   使用 mssql-conf 通过 set 命令更改默认数据目录

```
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultdatadir /mssql/data
```

　　现在，为新数据库创建的所有数据库文件都将存储在此新位置。 

　　d)   更改新数据库的日志文件 (.ldf) 位置，可以使用下面的 “set” 命令

```
mkdir -p /mssql/mssqllog
sudo chown mssql /mssql/mssqllog
sudo chgrp mssql /mssql/mssqllog
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultlogdir /mssql/mssqllog
```

　　e)   重启 SQL Server 服务

```
sudo systemctl restart mssql-server
```

更改默认转储目录位置
----------

　　使用 filelocation.defaultdumpdir 设置可更改每当系统崩溃时生成内存和 SQL 转储的默认位置。 默认情况下，这些文件在 /var/opt/mssql/log 中生成。

　　若要设置新位置，请使用以下命令：

　　a)   新的转储文件创建目标目录。 以下示例创建一个新的 **/mssql/mssql**dump 目录

```
sudo mkdir /mssql/mssqldump
```

　　b)   将目录的所有者和组更改为 **mssql** 用户

```
sudo chown mssql /mssql/mssqldump
sudo chgrp mssql /mssql/mssqldump
```

　　c)   使用 mssql-conf 通过 set 命令更改默认数据目录

```
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultdumpdir /mssql/mssqldump
```

　　d)   重启 SQL Server 服务

```
sudo systemctl restart mssql-server
```

启用可用性组 (默认不用开启)
---------------

　　使用 hadr.hadrenabled 选项可在 SQL Server 实例上启用可用性组。 下面的命令通过将 hadr.hadrenabled 设置为 1 来启用可用性组。 必须重启 SQL Server，该设置才能生效。

```
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled  1
sudo systemctl restart mssql-server
```

验证创建库
-----

　　a)   创建创建库的存储过程，如有这个存储过程就不用创建了。

```
CREATE PROCEDURE [dbo].[PROC_CREATE_DB]
    @DB_NAME  varchar(100),
    @data_path_root varchar(256) = 'D:\DBData\' --'/mssql/data/'
AS
BEGIN
    IF DB_ID (@DB_NAME) IS NOT NULL
    EXECUTE ('DROP DATABASE ' + @DB_NAME)
    
    -- execute the CREATE DATABASE statement 
    EXECUTE ('CREATE DATABASE ' + @DB_NAME + '
    ON 
    ( NAME = '''+ @DB_NAME +''',
        FILENAME = '''+ @data_path_root + @DB_NAME + '.mdf'',
        SIZE = 500,
        MAXSIZE = UNLIMITED,
        FILEGROWTH = 500 )
    LOG ON
    ( NAME = '''+ @DB_NAME +'_log'',
        FILENAME = '''+ @data_path_root + @DB_NAME + '_log.ldf'',
        SIZE = 50MB,
        MAXSIZE = UNLIMITED,
        FILEGROWTH = 50 )'
    )
    EXECUTE ('ALTER DATABASE ' + @DB_NAME  + ' SET RECOVERY SIMPLE')
END
```

　　b)   执行创建库的存储过程，注意路径

```
EXEC PROC_CREATE_DB ' 库名 ','/mssql/data/'  --库名、路径
```

查询当前时间
------

　　如果与当前时间不符，则需要修改系统时间：

```
clock --set --date="2020-10-19 19:30:39"
clock –hctosys
```

```
select GETDATE()
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111182602655-780618099.png)

　　如果一台服务器同时部署了 mysql，则修改时间后要去 mysql 查询当前时间

```
select now()
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111182634221-566024985.png)

创建用户并分配权限
---------

　　需要 SA 用户登录才有权限设置。

　　a)   应用程序和管理人员账号

 **Linux 上的 SQL Server 不支持 ADMINISTER BULK OPERATIONS 权限或 bulkadmin 角色。只有 sysadmin 可以在 Linux 上对 SQL Server 执行批量插入。**

 **sysadmin 读写权限比较高，专门给系统相关程序或管理员使用，不得通过任何人为方式使用。**

```
DECLARE @loginName VARCHAR(50) = ' 用户名 '
DECLARE @loginPassword VARCHAR(50) = ' 密码 '
IF EXISTS(SELECT 1 FROM sys.syslogins WHERE name = @loginName)
BEGIN
	PRINT ' 登录名【' + @loginName + '】已存在。'
	RETURN
END

DECLARE @sql VARCHAR(8000)
SET @sql = 'CREATE LOGIN ' + @loginName + ' WITH PASSWORD = ''' + @loginPassword + ''''
EXEC(@sql) 
--分配角色
EXEC sys.sp_addsrvrolemember @rolename = 'sysadmin', @loginame = @loginName
```

　　所以，如果是部署在 **Windows** 的话，程序账户用 bulkadmin、dbcreator。建议不用 sysadmin。

```
EXEC sys.sp_addsrvrolemember @rolename = 'bulkadmin', @loginame = @loginName
EXEC sys.sp_addsrvrolemember @rolename = 'dbcreator', @loginame = @loginName
```

删除用户（不要手贱！）
-----------

　　a)      在删除该登录名之前, 请更改相应数据库的所有者。

```
USE [SLSW_YN]; 
EXEC dbo.sp_changedbowner @loginame = N'sa', @map = false;
```

　　@map: 将别名及其权限移交给新的数据库所有者

　　找到对应用户所拥有的数据库权限，并转给其他用户, 如 SA 用户。

```
SELECT 'use ['+A.NAME+']; exec dbo.sp_changedbowner @loginame = N''sa'', @map = false; '
FROM SYS.DATABASES A
INNER JOIN SYS.SYSLOGINS B ON A.OWNER_SID=B.SID
WHERE B.NAME=' 用户名 '
```

　　b)   执行 a）产生的所有 SQL 语句

　   c)   杀掉账号所有线程再删除账号（不杀的话，禁用以后，原来打开的进程依然可以运行）

```
CREATE PROC [dbo].[PROC_mgr_login_process_kill_all]
@loginName VARCHAR(255)
AS
BEGIN
DECLARE @processes TABLE
(
ID INT IDENTITY(1, 1),
spid INT,
ecid INT,
status VARCHAR(50),
loginname VARCHAR(255),
hostname VARCHAR(255),
blk INT,
dbname VARCHAR(255),
cmd VARCHAR(8000),
request_id INT
)

DECLARE @sql VARCHAR(8000)
SET @sql = 'EXEC sp_who ''' + @loginName + ''''

INSERT INTO @processes
(
spid,
ecid,
status,
loginname,
hostname,
blk,
dbname,
cmd,
request_id
)
EXEC(@sql)

DECLARE @iLoop INT
DECLARE @totalCount INT

SELECT @iLoop = 1,
@totalCount = COUNT(*)
FROM @processes

WHILE @iLoop <= @totalCount
BEGIN
DECLARE @spid INT
SELECT @spid = spid FROM @processes WHERE ID = @iLoop

SET @sql = 'KILL ' + CAST(@spid AS VARCHAR(20))
EXEC(@sql)

SET @iLoop += 1
END
EN
```

```
EXEC MTNOH_AAA_DB.[dbo].[PROC_mgr_login_process_kill_all] ' 用户名 ';
EXEC sys.sp_droplogin @loginame = ' 用户名 ';
```

mssql.conf 格式配置
---------------

　　类似 mysql 的 etc/my.cnf

 **/var/opt/mssql/mssql.conf** 文件提供了每个设置的示例

```
cat /var/opt/mssql/mssql.conf
```

在线安装
====

自 CU20 起，SQL Server 2017 开始支持 RHEL 8。 以下用于 SQL Server 2017 的命令指向 RHEL 8 存储库。 RHEL 8 未预安装 SQL Server 所需的 python2。 在开始 SQL Server 的安装步骤之前，请执行以下命令，并验证是否选择了 python2 作为解释器：

```
sudo alternatives --config python
# If not configured, install python2 and openssl10 using the following commands:
sudo yum install python2
sudo yum install compat-openssl10
# Configure python2 as the default interpreter using this command:
sudo alternatives --config python
```

　　有关详细信息，请参阅以下博客，了解如何安装 python2 并将其配置为默认解释器： [https://www.redhat.com/en/blog/installing-microsoft-sql-server-red-hat-enterprise-linux-8-beta](https://www.redhat.com/en/blog/installing-microsoft-sql-server-red-hat-enterprise-linux-8-beta) 。

　　如果使用 RHEL 7，请将以下路径更改为 /rhel/7 而不是 /rhel/8。

安装 mssql-server
---------------

  下载 SQL Server 2017 Red Hat 存储库配置文件并安装

```
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
sudo yum install -y mssql-server
```

　　如果想安装 SQL Server 2019，必须改为注册 SQL Server 2019 存储库。 使用以下命令安装 SQL Server 2019：

```
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
sudo yum install -y mssql-server
```

配置
--

```
/opt/mssql/bin/mssql-conf setup
```

　　运行 mssql-conf setup，按照提示设置 SA 密码并选择版本。

安装 sqlcmd 和 bcp SQL Server 命令行工具
--------------------------------

　　下载 Microsoft Red Hat 存储库配置文件

```
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo
```

　　如果安装了早期版本的 **mssql-tools**，请删除所有旧的 unixODBC 包

```
sudo yum remove unixODBC-utf16 unixODBC-utf16-devel
```

　　使用 unixODBC 开发人员包安装 **mssql-tools**

```
sudo yum install -y mssql-tools unixODBC-devel
```

　…………

　　其他参考脱机安装章节

无人参与安装
======

```
#!/bin/bash -e

# Use the following variables to control your install:

# Password for the SA user (required)
MSSQL_SA_PASSWORD='<YourStrong!Passw0rd>'

# Product ID of the version of SQL server you're installing
# Must be evaluation, developer, express, web, standard, enterprise, or your 25 digit product key
# Defaults to developer
MSSQL_PID='evaluation'

# Install SQL Server Agent (recommended)
SQL_ENABLE_AGENT='y'

# Install SQL Server Full Text Search (optional)
# SQL_INSTALL_FULLTEXT='y'

# Create an additional user with sysadmin privileges (optional)
# SQL_INSTALL_USER='<Username>'
# SQL_INSTALL_USER_PASSWORD='<YourStrong!Passw0rd>'

if [ -z $MSSQL_SA_PASSWORD ]
then
  echo Environment variable MSSQL_SA_PASSWORD must be set for unattended install
  exit 1
fi

echo Adding Microsoft repositories...
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo

echo Installing SQL Server...
sudo yum install -y mssql-server

echo Running mssql-conf setup...
sudo MSSQL_SA_PASSWORD=$MSSQL_SA_PASSWORD \
     MSSQL_PID=$MSSQL_PID \
     /opt/mssql/bin/mssql-conf -n setup accept-eula

echo Installing mssql-tools and unixODBC developer...
sudo ACCEPT_EULA=Y yum install -y mssql-tools unixODBC-devel

# Add SQL Server tools to the path by default:
echo Adding SQL Server tools to your path...
echo PATH="$PATH:/opt/mssql-tools/bin" >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc

# Optional Enable SQL Server Agent :
if [ ! -z $SQL_ENABLE_AGENT ]
then
  echo Enable SQL Server Agent...
  sudo /opt/mssql/bin/mssql-conf set sqlagent.enabled true
  sudo systemctl restart mssql-server
fi

# Optional SQL Server Full Text Search installation:
if [ ! -z $SQL_INSTALL_FULLTEXT ]
then
    echo Installing SQL Server Full-Text Search...
    sudo yum install -y mssql-server-fts
fi

# Configure firewall to allow TCP port 1433:
echo Configuring firewall to allow traffic on port 1433...
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload

# Example of setting post-installation configuration options
# Set trace flags 1204 and 1222 for deadlock tracing:
#echo Setting trace flags...
#sudo /opt/mssql/bin/mssql-conf traceflag 1204 1222 on

# Restart SQL Server after making configuration changes:
echo Restarting SQL Server...
sudo systemctl restart mssql-server

# Connect to server and get the version:
counter=1
errstatus=1
while [ $counter -le 5 ] && [ $errstatus = 1 ]
do
  echo Waiting for SQL Server to start...
  sleep 5s
  /opt/mssql-tools/bin/sqlcmd \
    -S localhost \
    -U SA \
    -P $MSSQL_SA_PASSWORD \
    -Q "SELECT @@VERSION" 2>/dev/null
  errstatus=$?
  ((counter++))
done

# Display error if connection failed:
if [ $errstatus = 1 ]
then
  echo Cannot connect to SQL Server, installation aborted
  exit $errstatus
fi

# Optional new user creation:
if [ ! -z $SQL_INSTALL_USER ] && [ ! -z $SQL_INSTALL_USER_PASSWORD ]
then
  echo Creating user $SQL_INSTALL_USER
  /opt/mssql-tools/bin/sqlcmd \
    -S localhost \
    -U SA \
    -P $MSSQL_SA_PASSWORD \
    -Q "CREATE LOGIN [$SQL_INSTALL_USER] WITH PASSWORD=N'$SQL_INSTALL_USER_PASSWORD', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=ON, CHECK_POLICY=ON; ALTER SERVER ROLE [sysadmin] ADD MEMBER [$SQL_INSTALL_USER]"
fi

echo Done!
```

检查已安装的 SQL Server 版本
====================

　　若要验证 Linux 上的 SQL Server 的当前版本和版本，请先安装 SQL Server 命令行工具。

　　使用 “sqlcmd” 运行显示 SQL Server 版本的 Transact-SQL 命令。

```
sqlcmd -S localhost -U SA -Q 'select @@VERSION'
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111190108549-1608554281.png)

```
select @@VERSION
```

 ![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111190119184-1980255398.png)

```
SELECT SERVERPROPERTY('Edition')
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111190139420-1723849435.png)

卸载
==

　　若要删除 Linux 上的 “mssql-server” 包，不同平台命令会不一样：

| 

**平台**

 | 

**包删除命令**

 |
| 

**RHEL**

 | 

sudo yum remove mssql-server

 |
| 

SLES

 | 

sudo zypper remove mssql-server

 |
| 

Ubuntu

 | 

sudo apt-get remove mssql-server

 |

　　删除包不会删除生成的数据库文件。 如果希望删除数据库文件，请使用以下命令：

```
sudo rm -rf /var/opt/mssql/
```

更新或升级
=====

　　若要将 “mssql-server” 包更新到最新版本，请根据你的平台使用以下命令之一：

| 

**平台**

 | 

**包更新命令**

 |
| 

RHEL

 | 

sudo yum update mssql-server

 |
| 

SLES

 | 

sudo zypper update mssql-server

 |
| 

Ubuntu

 | 

`sudo apt-get update`  
`sudo apt-get install mssql-server`

 |

　　这些命令将下载最新包，并替换 `/opt/mssql/` 下的二进制文件。 此操作不会影响到用户生成的数据库和系统数据库。 

　　若要升级 SQL Server，请首先**将配置的存储库更改为所需的 SQL Server 版本**。 

　　然后使用同一个 update 命令升级 SQL Server 版本。  

 **存储库的作用**：用于获取数据库引擎包、mssql-server 以及相关 SQL Server 包。 

　　现有五个主要存储库：

| 

**存储库**

 | 

**名称**

 | 

**说明**

 |
| 

2019

 | 

mssql-server-2019

 | 

SQL Server 2019 累积更新 (CU) 存储库。

 |
| 

2019 GDR

 | 

mssql-server-2019-gdr

 | 

SQL Server 2019 GDR 存储库仅用于关键更新。

 |
| 

2019 预览版

 | 

mssql-server-preview

 | 

SQL Server 2019 预览版和 RC 存储库。

 |
| 

2017

 | 

mssql-server-2017

 | 

SQL Server 2017 累积更新 (CU) 存储库。

 |
| 

2017 GDR

 | 

mssql-server-2017-gdr

 | 

SQL Server 2017 GDR 存储库仅用于关键更新。

 |

回滚
==

　　若要将 SQL Server 回滚或降级到以前的版本，请使用以下步骤：

       a) 标识要降级到的 SQL Server 包的版本号。 有关包版本号的列表，请参阅[发行说明](https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-release-notes?view=sql-server-linux-2017)。

       b) 降级到 SQL Server 的早期版本。 在以下命令中，将 `<version_number>` 替换为步骤 1 中标识的 SQL Server 版本号。

| 

**平台**

 | 

**包更新命令**

 |
| 

RHEL

 | 

sudo yum downgrade mssql-server-<version_number>.x86_64

 |
| 

SLES

 | 

sudo zypper install --oldpackage mssql-server=<version_number>

 |
| 

Ubuntu

 | 

`sudo apt-get install mssql-server=<version_number>` sudo systemctl start mssql-server

 |

　　只支持降级到相同主版本（如 SQL Server 2017）内的版本。

功能支持情况
======

　　相比 Windows，Linux 会有些功能不完全支持。

| 

**类别**

 | 

**Windows**

 | 

**Linux**

 | 

**说明**

 |
| 

表

 | 

√

 | 

√

 |   |
| 

SQL 基础语法

 | 

√

 | 

√

 |   |
| 

存储过程

 | 

√

 | 

√

 |   |
| 

函数

 | 

√

 | 

√

 | 

包括 CLR 函数。不支持设置了 EXTERNAL_ACCESS 或 UNSAFE 权限的 CLR 程序集

 |
| 

索引

 | 

√

 | 

√

 |   |
| 

作业

 | 

√

 | 

√

 |   |
| 

视图

 | 

√

 | 

√

 |   |
| 

事务

 | 

√

 | 

√

 |   |
| 

数据库分区

 | 

√

 | 

√

 |   |
| 

链接服务器

 | 

√

 | 

√

 | 

只支持 SQLServer 链接服务器，不支持 Mysql。低版本连接高版本会出现部分问题

 |
| 

系统表

 | 

√

 | 

√

 |   |
| 

备份、还原

 | 

√

 | 

√

 |   |
| 

调用 bat

 | 

√

 | 

**×**

 | 

旧版本工单创建、更新会用到

 |
| 

BCP

 | 

√

 | 

**×**

 | 

xp_cmdshell、bulk insert，数据库入库会乱码

 |
| 

文件操作

 | 

√

 | 

**×**

 | 

xp_cmdshell、bulk insert，数据库入库会乱码

 |
| 

维护计划

 | 

√

 | 

**×**

 |   |

  
整库迁移
-------

　　把 Sqlserver2008（Windows）的数据库迁移到 Sqlserver2017（Linux）

 　a)  从 Sqlserver2008(Windows) 备份

```
SELECT 'BACKUP DATABASE ' + name + ' TO  DISK = N'''
+ 'D:'
+ '\' + name + '.bak''
   WITH COMPRESSION,NOFORMAT, NOINIT,
   NAME = N''' + name + '-完整 数据库 备份 '',
   SKIP, NOREWIND, NOUNLOAD,  STATS = 10'
FROM SYS.DATABASES
WHERE NAME IN (' 库名 1',' 库名 2')
ORDER BY NAME
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111190650422-480677405.png)

　　b)  在 Sqlserver2017(Linux) 还原

```
exec PROC_CREATE_DB ' 库名 ','/mssql/data/'
RESTORE DATABASE SLSW FROM DISK = '/home/slsw1009/SLSW.bak'
WITH
MOVE 'SLSW' TO '/home/mssqldata/SLSW.mdf',
MOVE 'SLSW_log' TO '/home/mssqldata/SLSW.ldf',
STATS = 1, REPLACE, RECOVERY
GO
```

![](https://img2020.cnblogs.com/blog/742073/202101/742073-20210111190844811-171940518.png)

 　　恢复数据库时出现 Exclusive access could not be obtained because the database is in use 错误，那是因为库正在使用。

　　运行以下 query 语句将数据库离线 (把 [db_name] 替换成你的数据库名，下同)

```
use master 
alter database [db_name] set offline with rollback immediate
```

　　接着进行数据库恢复的相关操作

```
RESTORE DATABASE SLSW FROM DISK = '/home/slsw1009//SLSW.bak'
WITH
MOVE 'SLSW' TO '/home/mssqldata/SLSW.mdf',
MOVE 'SLSW_log' TO '/home/mssqldata/SLSW.ldf',
STATS = 1, REPLACE, RECOVERY
GO
```

　　最后执行下面的 query 语句将数据库恢复在线

```
use master 
alter database [db_name] set online with rollback immediate;
```

 **c)  在 Sqlserver2017(Linux) 备份**

　　备份路径要有 mssql 用户的权限

```
mkdir -p /home/mssqlbackup
chown mssql /home/mssqlbackup
chgrp mssql /home/mssqlbackup
```

```
BACKUP DATABASE TestDB TO  DISK = N'/home/mssqlbackup/TestDB.bak'    
WITH COMPRESSION,NOFORMAT, NOINIT,    
NAME = N'TestDB_BACKUP', SKIP, NOREWIND, NOUNLOAD,  STATS = 10
```

已知问题
====

 a)   安装 SQL Server 的主机名的长度必须为 15 个字符或更少

解决方法：将 / etc / hostname 中的名称更改为 15 个字符以下的长度。

 b)   手动将系统时间倒退设置为时间会导致 SQL Server 停止更新 SQL Server 中的内部系统时间

解决方法：重新启动 SQL Server。

 c)   仅支持单实例安装

建议在主机上运行多个容器以具有多个不同的实例。使用 docker 可以轻松实现这一点，但是每个容器都需要侦听不同的端口

 d)   Linux 上不支持用户权限 ADMINISTER BULK OPERATIONS（bulkadmin 服务器角色）

 e)   SQLServer 和相关工具目前不支持在 Windows 10 上运行的 Linux

 f)   SQL Server 数据和日志目录不支持符号链接

 g)   重置系统管理（SA）密码

```
sudo systemctl stop mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sqlcmd -S myserver -U sa -P Test\$\$
```

如果在 SQL Server 登录密码中使用某些字符，则在终端的 Linux 命令中使用它们时，可能需要使用反斜杠将其转义。例如，在终端命令 / shell 脚本中使用美元符号（$）时，必须随时对其进行转义

[https://tldp.org/LDP/abs/html/special-chars.html](https://tldp.org/LDP/abs/html/special-chars.html)（特殊字符）

2017 新函数，让人眼前一亮的特性，有时间再补测试用例了。
==============================

　　Translate 函数（实现批量替换）

　　string_agg 函数（分组合并字符串）

　　trim 函数（移除左右空格、指定字符）

　　string_split 函数（拆分字符串）

　　数据转成 JSON 格式（2016 特性）

参考资料

[https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-configure-mssql-conf?view=sql-server-linux-2017](https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-configure-mssql-conf?view=sql-server-linux-2017%20（微软官方文档）%20https://www.cnblogs.com/shanyou/p/5272628.html%20) （微软官方文档）

[https://www.cnblogs.com/shanyou/p/5272628.html](https://www.cnblogs.com/shanyou/p/5272628.html) （背景段落摘抄）