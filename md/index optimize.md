> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/llgg/p/14102920.html)

*   **背景介绍：**

DBA 经常需要维护索引碎片保证查询效率，在维护索引的同时也会不可避免的造成部分语句的暂时 block。出于工作需要，编写了这个简单的索引维护逻辑。

*   **环境介绍**

sql server 2016 单节点 / alway on

*   **维护逻辑介绍**

入下图所示：1. 生成待维护的索引列表（monitor.dbo.dba_index_fragment）

                      2. 启动 job DBA_index_main 作为主进程 调用 job DBA_index_optimize_01/02/0* 并行维护索引。可根据需要创建不同数量的 job DBA_index_optimize_0* 来调节并行数量缩短 index 维护时间。

优缺点介绍：可以自动发现跳过长时间 block 其他 session, 继续维护队列内的下一个 index. 可以根据需要调节并行维护的进程数量。该逻辑仅仅维护 index 没有做统计信息更新

![](https://img2020.cnblogs.com/blog/569950/202012/569950-20201208145602241-2117368451.png)

创建完成后默认会有 3 个 job 入下图所示

![](https://img2020.cnblogs.com/blog/569950/202012/569950-20201208152955419-895011737.png)

*   **代码逻辑**

1.  database&log table: Monitor / dba_Log
2.  procedure: updba_index_fragment  /  updba_index_fragment_optimize  /  updba_index_main
3.  job: DBA_index_optimize_01  /  DBA_index_optimize_02  /  DBA_index_main

Monitor / dba_Log

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [master]
GO
create database monitor
go
use monitor
go
sp_changedbowner sa

use monitor
go
/*
drop table dba_Log
*/

create  table dbo.dba_Log
(
id bigint identity(1,1) primary key
,DatabaseName  nvarchar(200)
,Programe varchar(100)
,Command nvarchar(max)
,CommandType  varchar(200)
,StartTime  datetime
,EndTime datetime
,errorNumber  int
,errorMessage  nvarchar(max)
,note varchar(200)
)
create index ix_StartTime on dbo.dba_Log (StartTime)
```

View Code

procedure updba_index_fragment

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [Monitor]
GO
if ( object_id('dba_index_fragment') is not null
 and not exists(select  * from sys.columns with(nolock)  where object_id=OBJECT_ID('dba_index_fragment') and name='batch')
)    
    drop table dbo.dba_index_fragment
go


IF  EXISTS (SELECT * FROM monitor.sys.objects WHERE object_id = OBJECT_ID(N'dbo.updba_index_fragment') AND type in (N'P '))
    drop procedure dbo.updba_index_fragment
go

USE [Monitor]
GO

/*
get index fragment informations
edtior： lynn lin
run demo:
exec [dbo].[updba_index_fragment]
exec [dbo].updba_index_fragment 'dbname1'
exec [dbo].[updba_index_fragment]  'dbname1,dbname2','dbo.tbname1,dbo.tbname2'
select * from [dba_index_fragment] with(nolock)
truncate table dba_index_fragment
*/

create   proc [dbo].[updba_index_fragment]
    @db_list nvarchar(max) =  NULL   --NULL  all database ,   run database demo:  N'dbname1,dbname2' 
    ,@tb_list nvarchar(max) = NULL   --NULL  all table  ,     run table demo:  N'dbo.table1,dbo.table2'
    ,@fragment_mode varchar(10) ='LIMITED'     --LIMITED   SAMPLED  DETAILED  
    ,@SaveDays int=180 
as
begin
    set nocount on;

IF NOT EXISTS (SELECT * FROM monitor.sys.objects WHERE object_id = OBJECT_ID(N'monitor.dbo.dba_index_fragment') AND type in (N'U'))
        begin

            create  table dbo.dba_index_fragment
            (
            id int identity(1,1) not null primary key (id)
            ,batch int

            ,inTime datetime    default getdate()
            ,isop char(1)  default 'n'  --y need optimize  ， other don't need optimize
            ,optimize_spid int
            ,optimize_status varchar(15) default null --null, optmizing,canceled,completed
            ,opStartTime datetime
            ,opEndTime  datetime

            ,dbName    varchar(100)
            ,schemaName    varchar(50)
            ,tbName    varchar(100)
            ,ixName  varchar(500)
            ,ixType_desc  varchar(50)
            ,isClustered bit

            ,has_max_col bit
            ,allow_page_locks bit
            ,in_row_data_page_count int
            ,row_count  int


            ,avg_fragmentation_in_percent float 
            ,fragment_count  bigint
            ,page_count   bigint
            ,record_count bigint
            ,avg_record_size_in_bytes float


            ,user_seeks bigint
            ,user_scans bigint
            ,user_lookups bigint
            ,user_updates     bigint
            ,last_user_seek  datetime
            ,last_user_scan  datetime
            ,last_user_lookup datetime
            ,last_user_update datetime

            ,sqlcommand nvarchar(4000)
            ,comments nvarchar(max)

            ,dbId int
            ,tbId int
            ,ixId  int
            ,ixType tinyint 
            ,[DatabaseSpaceID] int
            ,[FileGroup]  nvarchar(50)
            ,[DatabaseFileName]  nvarchar(500)
            )
            create index ix_intime  on dba_index_fragment(intime)

        end
    else
        BEGIN
            delete monitor.dbo.dba_index_fragment where intime<dateadd(day,-(ABS(@SaveDays)),getdate())   --delete history data。
            --delete monitor.dbo.dba_index_fragment  where isop ='n'
        END


    --clear history data and useless data
    delete [monitor].[dbo].[dba_Log]
    where  programe='updba_index_fragment' and StartTime<dateadd(day,-(ABS(@SaveDays)),getdate())   --delete history data。



    declare @batch int
    select @batch=max(batch) from monitor.dbo.dba_index_fragment with(nolock)
    if @batch is null
        set @batch=1
    else
        set @batch=@batch+1
----fillter begin  ******************************
if object_id('tempdb.dbo.#dblist','U') is not null
    drop table #dblist
create table #dblist
(db varchar(400))

if object_id('tempdb.dbo.#tblist','U') is not null
    drop table #tblist
create table #tblist
(  db varchar(400)
    ,sche varchar(100)
    ,tb varchar(500)
)
--filter db
insert into #dblist(db)
select  T.name
from(

    SELECT
                name = PARSENAME(
                                LTRIM(RTRIM(T.c.value('.[1]', 'sysname'))),
                        )
            FROM(
                SELECT
                    database_name_list = CONVERT(xml,
                                                N'<c><![CDATA[' 
                                                + REPLACE(
                                                    REPLACE(
                                                        REPLACE(@db_list,CHAR(13),CHAR(10)
                                                        ),
                                                        CHAR(10),N','
                                                    ),
                                                    N',',N']]></c><c><![CDATA['
                                                )
                                                + N']]></c>'
                                            )
        
            )REQ
                CROSS APPLY REQ.database_name_list.nodes('/c/text()') T(c) 
        ) T
        inner join sys.databases d
            on T.name=d.name  and d.state=0
        WHERE T.name > N''


--filter tb
insert into #tblist(db,sche,tb)
SELECT 
db = ISNULL(PARSENAME(name, 3), N'*'),
sche = ISNULL(PARSENAME(name, 2), N'dbo'),
tb = PARSENAME(name, 1)
from(
            SELECT
                name = LTRIM(RTRIM(T.c.value('.[1]', 'sysname')))
            FROM(
                SELECT
                    table_name_list = CONVERT(xml,
                                            N'<c><![CDATA[' 
                                            + REPLACE(
                                                REPLACE(
                                                    REPLACE(
                                                        REPLACE(
                                                            @tb_list,
                                                            CHAR(9),
                                                            N'.'
                                                        ),
                                                        CHAR(13),
                                                        CHAR(10)
                                                    ),
                                                    CHAR(10),
                                                    N','
                                                ),
                                                N',',
                                                N']]></c><c><![CDATA['
                                            )
                                            + N']]></c>'
                                        )
            )REQ
                CROSS APPLY REQ.table_name_list.nodes('/c/text()') T(c)
    )T
    WHERE name > N''

--fillter end  ******************************


if object_id('tempdb.dbo.#dba_index_fragment','U') is not null
    drop table #dba_index_fragment
    create table  #dba_index_fragment
            (
            batch int

            ,inTime datetime    default getdate()
            ,isop char(1)   
            ,optimize_spid int
            ,optimize_status varchar(15) default null --null, optmizing,canceled,completed
            ,opStartTime datetime
            ,opEndTime  datetime

            ,dbName    varchar(100)
            ,schemaName    varchar(50)
            ,tbName    varchar(100)
            ,ixName  varchar(500)
            ,ixType_desc  varchar(50)
            ,isClustered bit


            ,has_max_col bit
            ,allow_page_locks bit
            ,in_row_data_page_count int
            ,row_count  int

            ,avg_fragmentation_in_percent float 
            ,fragment_count  bigint
            ,page_count   bigint
            ,record_count bigint
            ,avg_record_size_in_bytes float

            ,user_seeks bigint
            ,user_scans bigint
            ,user_lookups bigint
            ,user_updates     bigint
            ,last_user_seek  datetime
            ,last_user_scan  datetime
            ,last_user_lookup datetime
            ,last_user_update datetime

            ,sqlcommand nvarchar(4000)
            ,comments nvarchar(max)

            ,dbId int
            ,tbId int
            ,ixId  int
            ,ixType tinyint 
            ,[DatabaseSpaceID] int
            ,[FileGroup]  nvarchar(50)
            ,[DatabaseFileName]  nvarchar(500)
            )

    declare @sql_index nvarchar(max)
    select @sql_index=N'

;with COLMAX AS(
        SELECT C.object_id
        FROM sys.columns C WITH(NOLOCK)
        inner join sys.types T WITH(NOLOCK)
            on C.user_type_id = T.user_type_id
        WHERE(
                C.user_type_id IN(34, 35, 99) 
                OR(
                    C.user_type_id IN(165, 167, 231, 241)
                    AND C.max_length = -1
                )
            )
    )
insert into #dba_index_fragment(batch,dbId,tbId,ixId,ixType,dbName,schemaName,tbName,ixName,ixType_desc,has_max_col,
[allow_page_locks],in_row_data_page_count,row_count,DatabaseSpaceID,[FileGroup],[DatabaseFileName],isClustered
,user_seeks,user_scans,user_lookups,user_updates
,last_user_seek,last_user_scan,last_user_lookup,last_user_update,inTime,isop)
SELECT distinct  @batch,
dbId = db_id(), tbId=t.object_id, ixId=i.index_id, ixType = i.type
,dbName=db_name(), schemaName=sc.name, tb=t.name, ixName = i.name, ixType_desc = i.type_desc
,has_max_col=CASE WHEN EXISTS(
                            SELECT * FROM COLMAX WHERE object_id = t.object_id
                        )
                        THEN 1
                    ELSE 0
                END
,i.allow_page_locks,p.in_row_data_page_count,p.row_count 
,i.[data_space_id] AS [DatabaseSpaceID],f.[name] AS [FileGroup] 
,d.[physical_name] AS [DatabaseFileName]
,isClustered=case when i.type in ( 1,5 ) then 1 else 0 end

--index used info
,ixu.user_seeks,ixu.user_scans,ixu.user_lookups,ixu.user_updates    
,ixu.last_user_seek,ixu.last_user_scan,ixu.last_user_lookup,ixu.last_user_update,inTime = getdate() ,''n''
FROM  sys.tables t 
inner join sys.schemas sc with(nolock)
    on t.schema_id=sc.schema_id
inner join [sys].[indexes] i
    on i.object_id=t.object_id 
INNER JOIN [sys].[filegroups] f
    ON f.[data_space_id] = i.[data_space_id]
INNER JOIN [sys].[database_files] d
    ON f.[data_space_id] = d.[data_space_id]
INNER JOIN [sys].[data_spaces] s
    ON f.[data_space_id] = s.[data_space_id]
inner join sys.dm_db_partition_stats p  WITH(nolock)
    on p.object_id=i.object_id and p.index_id=i.index_id
left join sys.dm_db_index_usage_stats  ixu with (nolock)
        on i.object_id=ixu.object_id  and i.index_id=ixu.index_id 
WHERE 
t.is_ms_shipped=0    
and i.[type_desc] not in (  ''HEAP'', ''CLUSTERED COLUMNSTORE'')
and p.row_count >10000 and p.in_row_data_page_count>128
and (
        (    (ixu.user_seeks>100 or ixu.user_scans>100 or ixu.user_lookups>100)
            and ( ixu.last_user_seek>dateadd(DAY,-30,getdate())   or ixu.last_user_scan >dateadd(DAY,-30,getdate())  or ixu.last_user_lookup >dateadd(DAY,-30,getdate())  or ixu.last_user_update >dateadd(DAY,-30,getdate())  )
        )
        or ixu.object_id is null
)

ORDER BY t.object_id,i.index_id
    ,f.[name]
    ,i.[data_space_id]

if exists(select * from #tblist)
    begin
        delete  f
        from #dba_index_fragment f
        left join #tblist t
            ON t.sche=f.schemaName  and t.tb=f.tbName
        where t.tb is null
    end
declare @dbID int, @tbID int, @ixID int
DECLARE CUR_tb CURSOR LOCAL STATIC FORWARD_ONLY READ_ONLY
    FOR
    SELECT  dbID ,tbId, ixId
    FROM #dba_index_fragment with(nolock) where dbid=db_id()  order by  dbID ,tbId, ixId

    OPEN CUR_tb;
    FETCH CUR_tb INTO  @dbID, @tbID, @ixID
    WHILE @@FETCH_STATUS = 0
    BEGIN
        ;WITH FRG 
        AS( SELECT *
            FROM sys.dm_db_index_physical_stats(@dbID, @tbID, @ixID, NULL, @fragment_mode)
            --WHERE avg_fragmentation_in_percent > 30.0  and index_id > 0  and page_count>8
            )
        update f
            set page_count=FRG.page_count
                ,avg_fragmentation_in_percent=FRG.avg_fragmentation_in_percent
                ,fragment_count=FRG.fragment_count
                ,record_count=FRG.record_count
                ,avg_record_size_in_bytes=FRG.avg_record_size_in_bytes
                --,isop=''y''
        from #dba_index_fragment f with(nolock)
        inner join FRG
            on FRG.database_id=f.dbID  and FRG.object_id=f.tbID  and FRG.index_id=f.ixID

        update  #dba_index_fragment
        set isop=''y''
        where  avg_fragmentation_in_percent > 15.0  and ixId > 0  and page_count>8
            
        FETCH CUR_tb INTO  @dbID, @tbID, @ixID
    END
    CLOSE CUR_tb;
    DEALLOCATE CUR_tb;
'

if object_id('tempdb.dbo.#rundblist','U') is not null
    drop table #rundblist
create table #rundblist
(db varchar(400))

if exists (select *  from #dblist)
    begin
        INSERT INTO #rundblist
        SELECT distinct db from #dblist
    end
else
    insert into #rundblist
    SELECT name  FROM sys.databases DB WITH(NOLOCK)
    WHERE  name NOT IN('master', 'tempdb', 'model', 'msdb', 'distribution','monitor' )  and state =0  --online


--delete secondary db  for always on
;with SECONDARY_DB as
(
    select 
    adc.database_name as db
    /*
    ag.name GroupName,adc.database_name,rs.role_desc,cs.replica_server_name,
    ds.synchronization_state,ds.synchronization_state_desc,  --0 = NOT SYNCHRONIZING / 1 = SYNCHRONIZING / 2 = SYNCHRONIZED / 3 = REVERTING / 4 = INITIALIZING
    ds.synchronization_health,ds.synchronization_health_desc,   --0 = NOT_HEALTHY / 1 = PARTIALLY_HEALTHY  /  2 = HEALTHY
    ar.availability_mode,ar.availability_mode_desc,
    ar.failover_mode,ar.failover_mode_desc
    */
    from sys.availability_groups ag
    inner join sys.availability_replicas ar
        on ag.group_id=ar.group_id
    inner join sys.availability_databases_cluster adc
        on adc.group_id=ag.group_id
    inner join sys.dm_hadr_availability_replica_states rs
        on rs.group_id=ag.group_id and rs.replica_id=ar.replica_id
    inner join  sys.dm_hadr_availability_replica_cluster_states  cs
        on cs.group_id=ag.group_id and cs.replica_id=ar.replica_id
    inner join  sys.dm_hadr_database_replica_states ds
        on ds.group_id=ag.group_id  and ds.replica_id=ar.replica_id  and ds.group_database_id=adc.group_database_id 
    where role_desc='SECONDARY'  and cs.replica_server_name=@@SERVERNAME
)
delete #rundblist where db in (
    select * from SECONDARY_DB
)


    DECLARE CUR_DB CURSOR LOCAL READ_ONLY FORWARD_ONLY STATIC
    FOR
    SELECT db  FROM #rundblist 

    DECLARE
        @sql nvarchar(max),
        @database_name sysname;

    OPEN CUR_DB;
    FETCH CUR_DB INTO @database_name
    WHILE @@FETCH_STATUS = 0
    BEGIN;
        
        BEGIN TRY
        --log begin
         insert into monitor.dbo.dba_Log (DatabaseName,Programe  ,CommandType ,StartTime  ) select @database_name,'dba_index_fragment','index fragment',getdate()
         declare @identity bigint
         select @identity = null
         select @identity = SCOPE_IDENTITY()


            truncate table #dba_index_fragment
            SET @sql = N'USE ' + QUOTENAME(@database_name)+char(13)+ @sql_index
            EXEC sp_executesql @sql
                ,N'@fragment_mode sysname,@batch int'
                ,@fragment_mode,@batch
            insert into  monitor.dbo.dba_index_fragment
            select * from #dba_index_fragment where isop='y' order by dbID,tbId,isClustered DESC,page_count 

            --log end
            update monitor.dbo.dba_Log
            set EndTime=getdate()
            where id=@identity

        END TRY
        BEGIN CATCH
            --log begin and error
            update monitor.dbo.dba_Log
            set EndTime=getdate()
             ,errorNumber=ERROR_NUMBER()
             ,errorMessage=ERROR_MESSAGE()
            where id=@identity
            continue
            
        END CATCH;


        FETCH CUR_DB INTO @database_name
        
    END;
    CLOSE CUR_DB;
    DEALLOCATE CUR_DB;

    --BEGIN get optimize index scripts
    declare @isEdition int,@isOnline nvarchar(200)=' '
    select @isEdition=CASE
                        WHEN CONVERT(sysname, SERVERPROPERTY(N'Edition')) LIKE N'Enterprise%' THEN 1
                        ELSE 0
                    END
    if @isEdition=1
    begin
     SET @isOnline=N',ONLINE=ON'
    end


    
    declare @cpu_count int,@ncpu int
    select @cpu_count=cpu_count from  sys.dm_os_sys_info with(nolock)
    select @ncpu= 
        case   
            when @cpu_count<=4 then 1
            when @cpu_count<=8 then 2
            ELSE  4
        end


    --table have column index begin ####################################

    --* reorganize fragmentation<=30 and allow_page_locks=1 
    update dba_index_fragment
    set sqlcommand= ' use  '+QUOTENAME(dbName)+ ' alter index ' + QUOTENAME(ixName) + ' on ' + QUOTENAME(schemaName) + '.' + QUOTENAME(tbName) + ' reorganize '
    where  batch=@batch and  isop='y' and sqlcommand is null  and  ixType=5  
        and avg_fragmentation_in_percent<=30.0  and allow_page_locks=1  

    --* offline rebuild fragmentation>30
    update dba_index_fragment
    set sqlcommand= ' use  '+QUOTENAME(dbName)+ ' alter index ' + QUOTENAME(ixName) + ' on ' + QUOTENAME(schemaName) + '.' + QUOTENAME(tbName) + replace(' rebuild with(MAXDOP={@ncpu})','{@ncpu}',@ncpu)
    where batch=@batch and  isop='y' and sqlcommand is null and ixType=5


    --if table has colummn index, all indexs can't   rebuild  online
    update upt
    set sqlcommand= ' use  '+QUOTENAME(upt.dbName)+ ' alter index ' + QUOTENAME(upt.ixName) + ' on ' + QUOTENAME(upt.schemaName) + '.' + QUOTENAME(upt.tbName) + replace(' rebuild with(MAXDOP={@ncpu})','{@ncpu}',@ncpu)
    from dba_index_fragment upt
    join (
        select distinct dbId,schemaName,tbId from monitor.dbo.dba_index_fragment where batch=@batch and ixtype=5  
    ) tc
        on  tc.dbId=upt.dbId and tc.schemaName=upt.schemaName and tc.tbId=upt.tbId    
    where upt.batch=@batch and  upt.isop='y' and upt.sqlcommand is null 

    


    --table have column index  end #######################################


    --row index begin ####################################################

    --* reorganize,  fragmentation<=30 and allow_page_locks=1  
    update dba_index_fragment
    set sqlcommand= ' use  '+QUOTENAME(dbName)+ ' alter index ' + QUOTENAME(ixName) + ' on ' + QUOTENAME(schemaName) + '.' + QUOTENAME(tbName) + ' reorganize '
    where batch=@batch and isop='y' and sqlcommand is null and  ixType<>5  
        and avg_fragmentation_in_percent<=30.0  and allow_page_locks=1  
    

    --* rebuild offline ,   if table have text、ntext、image or FILESTREAM,  clustered index can't use rebuild online
    update dba_index_fragment
    set sqlcommand= ' use  '+QUOTENAME(dbName)+ ' alter index ' + QUOTENAME(ixName) + ' on ' + QUOTENAME(schemaName) + '.' + QUOTENAME(tbName) + replace(' rebuild with(FILLFACTOR = 100,MAXDOP={@ncpu})','{@ncpu}',@ncpu)
    where  batch=@batch and  isop='y' and sqlcommand is null  and  ixType<>5  
        and  isClustered=1   and has_max_col=1
    

    --* rebuild online
    update dba_index_fragment
    set sqlcommand= ' use  '+QUOTENAME(dbName)+ ' alter index ' + QUOTENAME(ixName) + ' on ' + QUOTENAME(schemaName) + '.' + QUOTENAME(tbName) + ' rebuild with(FILLFACTOR = 100'+@isOnline+replace(',MAXDOP={@ncpu})','{@ncpu}',@ncpu)
    where  batch=@batch and  isop='y' and sqlcommand is null  and  ixType<>5 
        

    --row index  end #########################################################

    delete [monitor].[dbo].dba_index_fragment
    where sqlcommand is null


end
GO
```

View Code

procedure updba_index_fragment_optimize

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [Monitor]
GO

IF  EXISTS (SELECT * FROM monitor.sys.objects WHERE object_id = OBJECT_ID(N'dbo.updba_index_fragment_optimize') AND type in (N'P '))
    drop procedure dbo.updba_index_fragment_optimize
go


USE [Monitor]
GO

/*
get index optimize command from table monitor.dbo.[dba_index_fragment] ,
editor： lynn lin
exec [updba_index_fragment_optimize] 
select * from [dba_index_fragment] with(nolock)
*/
create proc [dbo].[updba_index_fragment_optimize]
as
begin
set nocount on

    declare @sql nvarchar(4000),@id bigint,    @batch int
    select  @batch=max(batch) from  monitor.dbo.[dba_index_fragment] with(nolock)


     
    while 1=1
    begin
       
        select @sql = null, @id = null
        begin tran

        select  
            top(1) @id=id,@sql=sqlcommand  
        from  monitor.dbo.[dba_index_fragment] a   with(TABLOCKX)   --with(rowlock,xlock,readpast)  with(TABLOCKX)  
        where batch=@batch and isop='y' and  optimize_spid is null and optimize_status is  null 
        and not exists(
            select top(1) 1 from monitor.dbo.[dba_index_fragment] b with(nolock)
                where  b.batch=@batch and b.optimize_status='optimizing' and b.dbName=a.dbName  and b.schemaName=a.schemaName  and b.tbName=a.tbName
        )


        if @id is not null
            begin
                update monitor.dbo.dba_index_fragment
                set optimize_status='optmizing'
                    ,opStartTime=getdate()
                    ,optimize_spid=@@SPID
                where id=@id

                commit tran
            end
        else
            begin
                commit tran
                break
            end

            
        BEGIN TRY

            exec sp_executesql @sql

            update monitor.dbo.dba_index_fragment
            set optimize_status='completed'
                ,opEndTime=getdate()
                ,optimize_spid=-1
            where id=@id

        END TRY
        BEGIN CATCH

            update monitor.dbo.dba_index_fragment
            set optimize_status='error'
                ,comments = ERROR_MESSAGE()
                ,opEndTime=getdate()
                ,optimize_spid=-1
            where id=@id

        END CATCH;

    end
end


GO
```

View Code

procedure updba_index_main

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [Monitor]
GO

IF  EXISTS (SELECT * FROM monitor.sys.objects WHERE object_id = OBJECT_ID(N'dbo.updba_index_main') AND type in (N'P '))
    drop procedure dbo.updba_index_main
go

/**************************************************/
use monitor
go
/*
run procedure [updba_index_fragment] to optimize indexes
editor： lynn lin
exec [updba_index_main] 
select * from [dba_index_fragment] with(nolock)
*/
create  procedure updba_index_main
    @begin time='00:00:10'    --Maintenance start time
    ,@end  time ='23:59:10'   --Maintenance end time
as
BEGIN
    --updba_index_main
       declare 
        --@begin time='01:30:00'   --Maintenance start time
        --@end  time ='23:30:00'   --Maintenance end time
        @blocked_seconds int =120  --blocked fillter  IF blocked time>@blocked_seconds will skip to next
        ,@batch int
        ,@jobname sysname
        ,@id int

   --get fragment
    exec [dbo].[updba_index_fragment]

    --get index optimize job list
    declare @startjob table(  jobname sysname,flag int)
    insert into @startjob(jobname)
    select  name from msdb.dbo.sysjobs  where name like 'DBA_index_optimize_0%'

    --start index optimize job
    while 1=1
    begin
        select @jobname= null
        select top(1) @jobname=jobname from @startjob where flag is null
        if @jobname is null
            break
        EXEC [msdb].dbo.sp_start_job  @job_name = @jobname ; 
        waitfor delay '00:00:05'
        update @startjob
        set flag=1
        where jobname=@jobname
    
    end 


    --get curently batch
    select  @batch=max(batch) from  monitor.dbo.[dba_index_fragment] with(nolock)

    /*store job running status*/
     declare @job table
    (
        jobname sysname,
        spid int,
        id int,  -- column id  in table  monitor.dbo.[dba_index_fragment]
        flag int
    )

    while 1=1
    begin

        delete @job
        /*get cunrrent running job spid*/
        ;with jobinfo    
        as(
            select j.name as jobname,B.text,p.spid 
            from sys.sysprocesses p
            inner join  msdb.dbo.sysjobs j
                on p.program_name like '%'+replace( right(cast(j.job_id as varchar(50)),17),'-','') +'%'
            CROSS APPLY sys.dm_exec_sql_text(p.sql_handle) AS B
                where p.program_name like 'SQLAgent - TSQL JobStep%'  and  j.name like 'DBA_index_optimize_0%'
        )
        /*just get  optimize index spid*/    
        insert into @job(jobname,spid,id)
        select  jobinfo.jobname,jobinfo.spid,f.id 
        from jobinfo
        inner join monitor.dbo.[dba_index_fragment] f with(nolock) 
            on jobinfo.spid=f.optimize_spid  --and jobinfo.text=f.sqlcommand
        where f.batch=@batch


        --if not index oprimize job running will stop
        if not exists( select top(1) 1 from @job )
            break

        --if out of Maintenance windows stop
        if cast(getdate() as time) not between @begin and  @end   
        begin
                while 1=1
                begin

                    SELECT @jobname=null, @id=null
                    SELECT top(1) @jobname=jobname,@id=id from @job where flag is null
                    if @jobname is null 
                        break    
                    EXEC [msdb].dbo.sp_stop_job  @job_name = @jobname ;     
    
                    update monitor.dbo.[dba_index_fragment]
                    set optimize_spid=-1,    
                        optimize_status='cancled',
                        opEndTime=getdate()
                    where id=@id
    
                    update @job 
                    set flag=1
                    where jobname=@jobname                                    
                end

            break
        end
 
    /*get block job*/
    ;with blockespid as(
          select  
            distinct blocked from sys.sysprocesses  with(nolock) 
        where blocked<>0 and spid<>blocked and spid>50  and blocked>50
            and  waittime>(1000*@blocked_seconds)
      )
    delete @job
    where spid not in (
        select blocked from blockespid
    )


     while 1=1
     begin
        set @jobname=null
        set @id=null
        select top(1) @jobname=jobname,@id=id from @job where flag is null

        if @jobname is null 
            break

        EXEC [msdb].dbo.sp_stop_job  @job_name = @jobname ; 

        update monitor.dbo.[dba_index_fragment]
        set optimize_spid=-1,    
            optimize_status='cancled',
            opEndTime=getdate()
        where id=@id

        waitfor delay '00:00:10'
        EXEC [msdb].dbo.sp_start_job  @job_name = @jobname ; 

        update @job 
            set flag=1
        where jobname=@jobname

     end

     waitfor delay '00:02:00'

    end
END

/*****************************************************/
```

View Code

job DBA_index_optimize_01

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [msdb]
GO

/****** Object:  Job [DBA_index_optimize_01]    Script Date: 11/9/2020 2:34:11 PM ******/
BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0
/****** Object:  JobCategory [[Uncategorized (Local)]]    Script Date: 11/9/2020 2:34:11 PM ******/
IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'[Uncategorized (Local)]' AND category_class=1)
BEGIN
EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'[Uncategorized (Local)]'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

END

DECLARE @jobId BINARY(16)
EXEC @ReturnCode =  msdb.dbo.sp_add_job @job_name=N'DBA_index_optimize_01', 
        @enabled=1, 
        @notify_level_eventlog=0, 
        @notify_level_email=0, 
        @notify_level_netsend=0, 
        @notify_level_page=0, 
        @delete_level=0, 
        @description=N'No description available.', 
        @category_name=N'[Uncategorized (Local)]', 
        @owner_login_name=N'sa', @job_id = @jobId OUTPUT
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
/****** Object:  Step [DBA_index_optimize_01]    Script Date: 11/9/2020 2:34:11 PM ******/
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'DBA_index_optimize_01', 
        @step_id=1, 
        @cmdexec_success_code=0, 
        @on_success_action=1, 
        @on_success_step_id=0, 
        @on_fail_action=2, 
        @on_fail_step_id=0, 
        @retry_attempts=0, 
        @retry_interval=0, 
        @os_run_priority=0, @subsystem=N'TSQL', 
        @command=N'exec dbo.[updba_index_fragment_optimize]', 
        @database_name=N'Monitor', 
        @flags=0
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
COMMIT TRANSACTION
GOTO EndSave
QuitWithRollback:
    IF (@@TRANCOUNT > 0) ROLLBACK TRANSACTION
EndSave:
GO
```

View Code

job DBA_index_optimize_02

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [msdb]
GO

/****** Object:  Job [DBA_index_optimize_02]    Script Date: 11/9/2020 2:34:44 PM ******/
BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0
/****** Object:  JobCategory [[Uncategorized (Local)]]    Script Date: 11/9/2020 2:34:44 PM ******/
IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'[Uncategorized (Local)]' AND category_class=1)
BEGIN
EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'[Uncategorized (Local)]'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

END

DECLARE @jobId BINARY(16)
EXEC @ReturnCode =  msdb.dbo.sp_add_job @job_name=N'DBA_index_optimize_02', 
        @enabled=1, 
        @notify_level_eventlog=0, 
        @notify_level_email=0, 
        @notify_level_netsend=0, 
        @notify_level_page=0, 
        @delete_level=0, 
        @description=N'No description available.', 
        @category_name=N'[Uncategorized (Local)]', 
        @owner_login_name=N'sa', @job_id = @jobId OUTPUT
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
/****** Object:  Step [DBA_index_optimize_01]    Script Date: 11/9/2020 2:34:44 PM ******/
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'DBA_index_optimize_01', 
        @step_id=1, 
        @cmdexec_success_code=0, 
        @on_success_action=1, 
        @on_success_step_id=0, 
        @on_fail_action=2, 
        @on_fail_step_id=0, 
        @retry_attempts=0, 
        @retry_interval=0, 
        @os_run_priority=0, @subsystem=N'TSQL', 
        @command=N'exec dbo.[updba_index_fragment_optimize]', 
        @database_name=N'Monitor', 
        @flags=0
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
COMMIT TRANSACTION
GOTO EndSave
QuitWithRollback:
    IF (@@TRANCOUNT > 0) ROLLBACK TRANSACTION
EndSave:
GO
```

View Code

job DBA_index_main

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
USE [msdb]
GO

/****** Object:  Job [DBA_index_main]    Script Date: 11/9/2020 3:02:12 PM ******/
BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0
/****** Object:  JobCategory [[Uncategorized (Local)]]    Script Date: 11/9/2020 3:02:12 PM ******/
IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'[Uncategorized (Local)]' AND category_class=1)
BEGIN
EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'[Uncategorized (Local)]'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

END

DECLARE @jobId BINARY(16)
EXEC @ReturnCode =  msdb.dbo.sp_add_job @job_name=N'DBA_index_main', 
        @enabled=1, 
        @notify_level_eventlog=0, 
        @notify_level_email=0, 
        @notify_level_netsend=0, 
        @notify_level_page=0, 
        @delete_level=0, 
        @description=N'No description available.', 
        @category_name=N'[Uncategorized (Local)]', 
        @owner_login_name=N'sa', @job_id = @jobId OUTPUT
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
/****** Object:  Step [updba_index_main]    Script Date: 11/9/2020 3:02:13 PM ******/
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'updba_index_main', 
        @step_id=1, 
        @cmdexec_success_code=0, 
        @on_success_action=1, 
        @on_success_step_id=0, 
        @on_fail_action=2, 
        @on_fail_step_id=0, 
        @retry_attempts=0, 
        @retry_interval=0, 
        @os_run_priority=0, @subsystem=N'TSQL', 
        @command=N'
exec  updba_index_main', 
        @database_name=N'Monitor', 
        @flags=0
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
COMMIT TRANSACTION
GOTO EndSave
QuitWithRollback:
    IF (@@TRANCOUNT > 0) ROLLBACK TRANSACTION
EndSave:
GO
```

View Code

*   **以下内容介绍如何根据需要个性化定制：**

1. 维护时间窗口设置：

![](https://img2020.cnblogs.com/blog/569950/202012/569950-20201209101506390-1188061787.png)

2. 只维护特定的 db， 特定的 table，  获取索引碎片参数，历史记录保存时间

![](https://img2020.cnblogs.com/blog/569950/202012/569950-20201209101618522-1122307717.png)

 3. 修改 procedure [updba_index_fragment] 逻辑过滤不需要维护的索引

![](https://img2020.cnblogs.com/blog/569950/202012/569950-20201209102242257-1418687482.png)