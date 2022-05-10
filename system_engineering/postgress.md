# postgres

## installation

### 编译

    ./configure --prexfx=/usr/local/pgsql13.2 --with-perl --with-python
  
    make
  
    makeinstall
  
### 查看编译参数

    pg_config
  
### contrib

    cd postgres-13.2/contrib
    make
    make install

### 起停

    export PGDATA=~/working/pgdata/
    
    initdb
      initdb -k : 打开 checksum 打开 checksum 功能
    
    pg_ctl start -D $PGDATA
    
    pg_ctl stop -D $PGDATA [-m MODE]  
    
    mode:
     - smart
     - fast
     - immediate
     
## 基本配置

    pg_hba.conf :  Client Authentication Configuration File  
        修改允许远程连接
        
    postgresql.conf : 主配置文件  
    ```
        # 监听地址和端口
        listen_addresses = '*' 
        port = 5432
        
    ```
    
### 日志方案

    ```
    # 1. 每天生成新文件
        log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
        log_truncate_on_rotation = off
        log_rotation_age = 1d			
        log_rotation_size = 0

    # 2. 写满 10M 生成新文件
        log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
        log_truncate_on_rotation = off
        log_rotation_age = 0			
        log_rotation_size = 10MB		

    # 3. 默认, 只保留7天
        log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
        log_truncate_on_rotation = on
        log_rotation_age = 1d
        log_rotation_size = 0
        
    ```

    
### 一些性能相关

    shared_buffers = 256MB  
    work_mem = 8MB  
    max_connections = 100
    
    
#### 数据块

    ./configure --with-blocksize=32
                --with-wal-blocksize=32
                --with-wal-segsize=64
                
                
    

## 基本操作

    ```
      \l , 查看所有数据库
      \c 数据库名 , 连接到数据库
      \d , 查看数据库表
      \d 表名, 查看数据库表结构
      \d t_pkey, 索引信息
      \d+ xxx, 更多信息
      
      匹配
      \dt, 表
      \di, 索引
      \ds, 序列
      \dv, 视图
      \df, 函数
      
      \timing on , 显示时间

      \dn , 列出所有schemas
      \db , 表空间
      \dg , 用户
      \du , 用户
      \dp , 权限分配
      \z  , 权限
      
      \encoding , 设置客户端字符集
      
      \pset , 设置输出格式
               \pset booder 2 , 边框
               \pset format unaligned , 对齐
               \pset fieldsep '\t', 分隔符
               
      \o , stdout
      \x , 按列展示
      \i , 执行外部sql
      \e , 编辑器
           \setenv PSQL_EDITOR "emacs"

      
  
    ```




## 类型

    - 布尔
    - 数值
    - 字符串
    - 二进制
    - 位串
    - 日期时间
    - 枚举
    - 几何
    - 网络地址
    - 数组
    - 复合类型
    - xml 
    - json
    - range
    - 对象标识
    - 伪类型: any 等等
    - 其他: 

### 类型转换

    select CAST('5' as int)
    select '5'::int 
    
### 复合类型

    ``` 
    # create TYPE person AS(
    # name text,
    # age integer,
    # sex boolean
    # );
    
    CREATE TYPE
    
    ```
    
    
    '(val, val,...)'
    '(val,,...)'    第二个为NULL
    '(val,"",...)'    第二个为空    
    
    访问时,使用点 `.` , 
    

### xml 类型

    编译时, 带上 `--with-libxml`,
    
    ```
    select xml '<hello>helloworld</hello>'
    ```


    show xmloption;
    

## 逻辑解构管理

### 介绍
    
    数据库: 
    表,索引: relation
    数据行: Tuple
    
### 基本操作

    创建: 
    
      create database osdbadb;
      
    参数:
    
      OWNER=user_name , 不指定为当前用户
      TEMPLATE=template, 模板名, 默认 template1,  0 可作为创建任意字符集的模板,是不包含任何受字符集编码或排序影响的数据或索引
      ENCODING=UTF8, 
      TABLESPACE=tablespace, 关联表空间
      CONNECTION LIMIT=num, 并发连接,默认-1, 无限
      
    修改:
    
        ALTER DATABASE name RENAME TO new_name  
        ALTER DATABASE name OWNER TO new_owner  
        ALTER DATABASE name SET TABLESPACE new_ts  
        
        
    删除:
    
        DROP DATABASE [ IF EXISTS ] name
        
        如果还有连接在这个数据库上, 无法删除
        
### 模式

    模式(Schema), 可以理解为一个命名空间或目录,
    不同的模式可以有相同的表, 函数等对象而不会冲突.
    
    一个数据库包含一个或多个模式.  
    类似mysql中的database, pg中,访问不同的数据库要重新连接, 模式没有此限制.
    
    特性:  
    - 允许多个用户使用同一个数据库, 互相不干扰.
    - 把数据库对象放到不同的模式下组成逻辑组,便于管理.
    - 第三方应用可以放到不同模式中, 不会产生名字冲突.
    
    创建:  
        CREATE SCHEMA schema_name [ AUTHORIZATION username ] [ schema_element ...]
        CREATE SCHEMA AUTHORIZATION username [ schema_element ... ]
        
        创建模式的同时,还可以在下面创建表的视图  

        
    更改:  
        ALTER SCHEMA name RENAME TO new_name
        
    访问:  
        默认会创建一个 public 的模式, 访问时不需要指定
        
        schema_name.table_name
        
    搜索:  
        
        ```
        guo=# show search_path;
        search_path   
        -----------------
        "$user", public
        (1 row)

        ```
    
    权限:  
        默认情况下, 用户不能访问不属于自己模式中的表, 
        需要时要赋予 "USAGE" 权限.
        
        ```
        REVOKE CREATE ON SCHEMA public FROM PUBLIC;

        public: 模式名
        PUBLIC: 所有用户
         
        ```
        
    兼容:  
        Oracle中, 一个用户对应一个schema;  
        建议删除public模式;  
        mysql中没有模式, 移植时, 建议每个mysql 数据库对应一个模式
        
### 表

    创建:  
        CREATE TABLE table_name (...);
        
    主键:  
        单一主键:  
        CREATE TABLE table_name (id int primary key, ...) ;
        复合主键:  
        CONSTRAINT constraint_name PRIMARY KEY (col1, col2);
        
    约束:  
        CONSTRAINT constraint_name UNIQUE (col1, col2);
        CONSTRAINT constraint_name CHECK (expression);
        
    根据模板创建:  
        CREATE TABLE baby(LIKE child);
        CREATE TABLE baby(LIKE child INCLUDE ALL);
        
#### 表的存储属性

    TOAST： the oversized-attribute storage technique，
        主要用于大字段的值。
        
        pg页面大小固定，通常8k， 不允许跨多个页面，
        为了突破此限制， 大字段通常被压缩或切成多个物理行存到一个系统表中，
        即 TOAST 表.
        
        如果一个表中有任何一个字段可以 TOAST ，pg会创建一个 TOAST 表，
        OID 存储在表的 pg_class.retoastrelid 中， 
        
        行外存储被切成若干 chunk 块，每个 chunk 块是一个block的四分之一，
        块8k， chunk 2k， 每个 chunk 作为独立的行在 TOAST 表中存储。
        chunk_id, chunk_seq, chunk_data
        
    存储属性：
        toast_tuple_target
        toast.fillfactor
        
#### 临时表
    
    会话临时表：  
        create TEMPORARY table tmp （id int  primary key， not text);

    事务级临时表：  
        create TEMPORARY table tmp （id int  primary key， not text) on commit delete rows;        
    
    二者都会在会话结束时消失。
    
    临时表在特殊的 schema 中，
        
#### unlogged 表

    没有 wal 的表， 无法主从同步， 提高了写入性能。
    
    create unlogged table
    
#### 表继承

    create table xxx (...) INHERITS xxx
    
    可以使用表继承来分区; 通常配合触发器进行数据的插入.

        
### 触发器

    对表进行 insert, update, delete 时, 可触发执行.
    
    创建触发器通常两步:
      - 创建一个执行函数
      - 创建一个触发器
      
    两种触发器:
      - 语句级: FOR STATEMENT； 记录实际未执行的操作
      - 行级:   FOR EACH ROW； 不记录未执行的操作
      
      
    \df : 查看触发器
    \sy : 查看函数定义
    
    
    两个时间:
      - before :  可直接修改 new 值, 以改变实际的更新结果.
      - after  : 
      
    
    删除:
        DROP TRIGGER [ IF EXISTS ] name ON table [ CASCADE | RESTRICT ];
        
        要指定 on table
    
    
    返回值:
        语句级, 需要显示指定 return NULL
        
        BEFORE | INSTEAD OF ,行级, 返回NULL ,忽略当前操作, 返回非NULL, 即为插入和更新的行
        AFTER, 返回值将被忽略
        

    event trigger
    
### 表空间

    不同存储或者fs
    
    \db : list 表空间
    
    
### 视图    

### 索引

    - BTree
    - HASH:    更新不会记录在wal
    - GiST:    一种架构
    - SP-GiST: 
    - GIN :    反转索引, 处理包含多个键值, 如数组
    
    
    ```
    create table contacts(
        id int primary key,
        name varchar(40),
        phone varchar(32)[],
        address text
    );
    
    create index idx_contacts_phone on contacts using gin(phone);
    
    select * from contacts where phone @> array['13422334455'::varchar(32)];
    
    ```
    索引可增加存储参数: 
      fillfactor : https://www.yisu.com/zixun/256414.html
      
    无阻塞创建:
      concurrently,  
      
    更改:  
    
        alter index
    
    删除:  
        drop index 
        
        
### 用户及权限

    创建用户和角色没区别, 除了用户可以登录以外;


### 事务,并发,锁

    ACID :
    
    MVCC , 多版本并发, 读不阻塞写, 写不阻塞读
    
    DDL事务,  与其他数据库不同的是,  大多数DDL也可以包含在事务当中, 且可以回滚.
    
    默认事务开启:   
        \set AUTOCOMMIT off
        \echo :AUTOCOMMIT 
        
        还可以通过 `BEGIN;` 启动一个事务
        
    保存点:
        SAVEPOINT xx01
        rollback to SAVEPOINT xx01
        
    隔离级别:
        - read uncommitted 读为提交
        - read committed   读已提交
        - repeatable read  重复读
        - serializable     串行化
        
        脏读
        不可重复读
        幻读
        
    两阶段提交:
    
        prepare transaction 'xxxid'
        commit prepared 'xxxid'
    
    锁机制:
    
## PostgreSQL 核心架构

### 进程

    postmaster:   
      postgres 子进程  
      辅助子进程:  
          - Logger
          - BgWritter
          - PgArch, 归档
          - PgStat, 统计信息收集
          - Auto Vacuum，系统自动清理
          - WalWriter
          - Checkpoint

#### 进程介绍

    主进程 postmaster 即postgres， 客户端每个连接会创建一个进程，
    可通过查询 pg_stat_activity 查看服务进程
       
#### 共享内存

### 存储结构  

    逻辑结构：
        数据库簇 -> datbase -> schema -> 表,外部表,view, 索引，函数  
        
    物理结构：  

    ```
        默认表空间目录/database oid/relfilenode.顺序号  
        
        表空间根目录/catalog version 目录/database oid/relfilenode.顺序号  
    ``` 
    
    pg_database
    pg_class
    
    ```
    查看
        select relnamespace, relname, relfilenode  from pg_class where relname='student';
    ```
    
### 应用程序访问接口

## 服务管理

### 服务启停

    - postgres 运行
    - pg_ctl 运行
    
    ```
        postgres -D $DATA &
        
        pg_ctl -D $data start
    ```
    
    发送信号停止:
      - SIGTERM : Smart shutdown 
      - SIGQUIT: Fast shutdown
      - SIGOUT: Immediate shutdown
      
    ```
    pg_ctl
    
    pg_ctl stop -D DATADIR -m smart
    pg_ctl stop -D DATADIR -m fast
    pg_ctl stop -D DATADIR -m immediate
    
    ```
    
    pg_ctl 是使用工具集, 包括以下功能
    - init 初始化
    - start, stop ,起停
    - 查看状态
    - 重新读取配置文件
    - 允许给指定进程发信号
    - 在windows 平台允许为数据库实例注册或取消一个系统服务
    
    ```
    pg_ctl relead -D $DATA
    
    pg_ctl status -D $DATA
    
    pg_ctl kill xxsig processid
    
    select pg_backend_pid();
    
    ```
    
    
    单用户模式:  
      参数 --single  
      主要用于以下场景:
        - 多用户不接受命令
        - initdb 阶段
        - 修复系统表
        
        
      ```
          postgres --single -D xxx postgres
          vacuum full
      ```

### 参数

    可以使用 include 'filename' 包含其他配置文件.
    
    pg_settings 表  
    其中 context 表示配置项的种类
    
      - internal, 通常是只读参数, 写死的, 初始化后无法变更
      - postmaster, 改变后需要重启
      - sighup, 改变后不需要重启, 只需要发送 SIGHUP 信号重新装载
      - backend, 无需重启, 但新配置只会出现在修改之后的新连接中
      - superuser, 由超级用户使用 SET 改变, 只影响 该session,
      - user, 由普通用户使用 SET 改变
    
#### 连接配置项

    max_connections: 最大连接数;  
        与操作系统的信号量有关, kernel.sem； 
        要考虑到local memory，尤其是work_mem； 
        考虑应用连接池, 也可以y使用应用和数据库中间层, 例如 pgbouncer
        
#### 内存配置项

    shared_buffers :  通常, 可设置为系统内存的 25%
    tmp_buffers :
    work_mem : 
    maintenance_work_mem :
    max_stack_depth : 执行堆栈的最大安全深度, 通常不变；当复杂函数无法运行考虑调整
    
#### wal 配置项

    wal_level :  物理备库, 需要replica, 逻辑同步, 设置为logical
    fsync : 是否使用fsync() 系统调用;  正常为on, 为了提高性能,非产品数据库可以关掉
    synchronous_commit : 提交事务时, 是否需要等待wal 刷盘；提交不重要事务可关闭提高性能
    wal_sync_method : 
    full_page_writes : 默认打开, 在checkpoint 之后,对页面进行第一次修改时, 将整页写入wal
    wal_buffers : 
    wal_writer_delay : 通常在 synchronous_commit off 时有用, on时每次提交都会刷盘
    commit_delay : wal 缓冲刷盘的延迟, 
    commit_siblings : 执行 commit_delay 延时需要的最小并发事务

#### 错误报告和日志项

    log_destination : 可以 syslog 或 eventlog, 还可以csv log
    
    log_min_duration_statement : 慢查询记录
    
    log_checkpoints : 有各种日志记录的开关
    
### 访问控制    

    pg_hba.conf
    
### 备份和还原

#### 逻辑备份    
    
    pg_dump, pg_dumpall 命令对数据库逻辑备份;  
    pg_dump 不阻塞其他用户的访问    
    可生成脚本文件和归档文件  

    pg_restore
    
#### 物理备份

    - 冷备份  
      停数据库, copy 数据库 pgdata 目录
    - 热备份   
      使用 PITR  
      使用文件系统快照  
      lvm 做快照, mount 快照, tar, copy 到另一台机器解压

### 常用命令

#### 查看
    ```
        查看版本
            select version()
        
        查看启动时间
            select pg_postmaster_start_time()
        
        查看data 目录    
            show data_directory;
    
        查看系统表  
            \dt pg_*
            
        查看最后load 配置文件的时间
            select pg_conf_load_time()
            
        查看时区
            show timezone
        
        查看时间
            show now()
            
        查看数据库
            pg_sql -l
            \l
            
        查看用户
            show user
            show current_user
            show session_user
            
            set role u01
            show session_user 时, 与user 不同;
              session_user 是原始用户,user 是当前用户
              
        查询当前连接数据库的名称
            select current_catalog,current_database();
        
        查询IP和端口
            select inet_client_addr(), inet_client_port();
            select inet_server_addr(), inet_server_port();
            
        后他进程id
            select pg_backend_pid();
            
        查看配置
            show shared_buffers;
            select current_setting('shared_buffers');
            
        修改当前 session 参数配置            
            set maintenance_work_mem to '128MB'
            select set_config("maintenance_work_mem", '128MB', false);
            
        查看wal
            select  pg_xlogfile_name(pg_current_xlog_location());
            ???
            
        查看是否正在做备份
            select pg_is_in_backup(), pg_backup_start_time();
        
        查看是否出于 hot standby 状态
            select pg_is_in_recovery();
            
        查看数据库大小
            select pg_database_size('guo'), 
              pg_size_pretty(pg_database_size('guo'))
              
        查看表大小
            select pg_size_pretty(pg_relation_size('student'));
            含索引
            select pg_size_pretty(pg_total_relation_size('student'));
            仅索引
            select pg_size_pretty(pg_indexes_size('student'));
            表空间
            select pg_size_pretty(pg_tablespace_size('pg_default'));
            
        路径
            select pg_relation_filepath('student');
    ```
       
#### 维护
    
    ```
        配置
            pg_ctl reload
            select pg_reload_conf();
        
        切换日志
            select pg_rotate_logfile();
            
        切换wal
            select pg_switch_xlog();
            
        产生一个checkpoint
            checkpoint
            
        取消长时间执行的sql
            pg_cancel_backend(pid);
            pg_terminate_backend(pid);
            
            先找出长时间 sql
                select pid, usename, query_start, query from pg_stat_activity;
            
    ```

## 执行计划

    EXPLAIN options statement
    
    options:
      - ANALYZE boolean : 
      - VERBOSE boolean : 附加信息
      - COST boolean : 启动成本和总成本
      - BUFFERS : 缓冲区使用信息
      - FORMAT : 输出格式, text, xml, json, yaml
      
      
      ```
      # explain select * from student;
                         QUERY PLAN
      ------------------------------------------------------------
      Seq Scan on student  (cost=0.00..16.00 rows=600 width=106)
        (1 row)
        
     seq scan: 顺序扫描, 即全表扫描
     cost : 启动成本..返回数据的成本
     rows : 返回行数
     width : 每行平均宽度
     
      ```
      顺序扫描一个数据块: cost 值为1  
      随机扫描一个数据块: cost 为4  
      处理一个数据行cpu代价, cost 为 0.01
      处理一个索引行cpu代价, cost 为 0.005
      每个操作符的cpu 代价, 0.0025
      
      加上 analyze 后 , 可以看到实际的时间和成本
      
      ```
      explain analyze select * from student;
                                              QUERY PLAN                  
      ------------------------------------------------------------------------------------------------------
      Seq Scan on student  (cost=0.00..16.00 rows=600 width=106) (actual time=0.008..0.010 rows=5 loops=1)
      Planning Time: 0.050 ms
      Execution Time: 0.025 ms
      (3 rows)
      
      只看路径
      # explain (costs false)  select * from student;
          QUERY PLAN      
      ---------------------
      Seq Scan on student
      (1 row)

      explain (analyze true, buffers true)  select * from student;
                                              QUERY PLAN
      ------------------------------------------------------------------------------------------------------
      Seq Scan on student  (cost=0.00..16.00 rows=600 width=106) (actual time=0.007..0.009 rows=5 loops=1)
      Buffers: shared hit=1
      Planning Time: 0.040 ms
      Execution Time: 0.019 ms
      (4 rows)
      
      ```

      加了 buffer后,
      shared hit 表示共享内存中直接读到了 xxx 个块
      read 表示磁盘读到了 xxx 个块
      written 表示写了 xxx个块


### 扫描
    
    顺序扫描

    索引扫描
    
      ```

       #explain select * from student where no = 1;
                                  QUERY PLAN
       ------------------------------------------------------------------------------
       Index Scan using student_pkey on student  (cost=0.15..8.17 rows=1 width=106)
       Index Cond: (no = 1)
       (2 rows)

      ```



    位图扫描:   
        扫描索引, 把满足条件的行或块在内存中建立位图, 扫完索引后, 根据位图到表的数据文件中把数据读出来.  
        非等值查询, IN 子句, 或多个条件都可以走位图扫描
        
       ```
       # explain select * from student where no in ( 1,2);
                                QUERY PLAN                                 
       ---------------------------------------------------------------------------
       Bitmap Heap Scan on student  (cost=8.32..13.66 rows=2 width=106)
           Recheck Cond: (no = ANY ('{1,2}'::integer[]))
           ->  Bitmap Index Scan on student_pkey  (cost=0.00..8.32 rows=2 width=0)
           Index Cond: (no = ANY ('{1,2}'::integer[]))
       (4 rows)

       ```
            
            
    条件过滤:   
        Filter :  xxx  
        
### 连接

    循环嵌套连接 : Nestloop join   
        一个驱动表(outer table), 一个 Inner table, 
        驱动表中每一行与inner 表中的检索找到与他匹配的行.
        返回数据不宜过大, 把返回子集较小的作为外表
    
    散列连接:  hash join  
        较小的表在内存中建立散列表, 扫描较大的表, 匹配响应的行
        
    合并连接: merge join  
    


### 执行计划相关配置项

    ENABLE_*,  可以临时处理一些执行计划不理想的情况;  
      通常情况下, pg不会选错, 如果与理想不一致,可频繁使用 analyze 更新所需信息.
      
    cost 基准,  
      通常,基准为顺序扫描一个数据块, `seq_page_cost` 为 1
      
    基因查询优化参数,  
      GEOQ 是使用探索式搜索来执行查询规划的算法, 可缩短负载查询的规划时间.
      该检索是随机, 生成的计划有不确定性.
      
### 统计信息的收集


    由 Auto Vacuum 进程进行收集 , 用于查询优化时的代价估算.
    
    表和索引的行数, 记录在 `pg_class`, 其他记录在 `pg_statistic` 中.
    
    Status Colletcor 子进程是 pg中专门收集性能统计信息的.
    可以通过 pg_stat_* 视图进行查看.
    
    日志控制:  
      - log_statement_stats
      - log_parser_stats
      - log_planner_stats
      - log_executor_stats
      
    手动收集:  
    
      ANALYZE 命令用于收集统计信息, 存储在 pg_statistic 表中.
      Auto Vacuum 默认开启,关闭时, 可每天在数据库空闲时, 
      运行一次 vacuum 和 analyze
      
## 技术内幕

### 隐含字段

    每个表有一些隐含字段:  
      - oid :: 行对象标识符, 在建表时, 使用"with oids" 或配置"default_with_oid时出现
      - tableoid :: 包含本行的表的oid.
      - xmin :: 插入该行版本的事务 ID
      - xmax :: 删除此行时的事务 ID, 第一次插入时为0, 如果不为0, 说明删除该行事务未提交或回滚了
      - cmin :: 事务内部的插入类操作命令ID, 从0开始
      - cmax :: 事务内部的删除类操作命令ID
      - ctid :: 一个行版本在他所处的表的位置
      
      
#### ctid

    ctid 表示数据行在表中的物理行数.
    
    ```
    # select ctid, no from student;
      ctid  | no  
     -------+-----
      (0,1) |   1
      (0,3) | 100
      (0,4) | 101
      (0,5) | 102
      (0,6) | 103
     (5 rows)
    ```
    
    0 表示所在物理块号, 另一个数字表示数据行在物理块中的行号.
    
    可利用 ctid 删除重复数据.
    
#### xmin xmax cmin cmax

    新插入一行, xmin 为当前事务ID, xmax 为0
    修改一行, 实际上是新插一行, 老数据行 xmin不变 ,xmax 为事务id, 新数据行上的xmin为当前事务id, xmax 为0
    删除一行, 把当前行的xmax 为当前事务id
    
    由此, 没有修改行的操作, 修改事实上相当于是将原数据行xmax 打上删除标记.
    
    
    游标相当于快照, 无法看到游标引用后数据的变化;
    pg使用了与事务内可见性问提类似的方法, 引入了命令id.
    当命令读取这行数据时, 如果当前命令id大于等于数据行上的命令id, 说明数据可见.
    如果当前命令小于数据行上的命令id, 说明不可见.
    
### 多版本并发 MVCC

    通过记录数据的状态 Commit log(clog), 可知道事务的内数据是否有效.
    
    四种状态:
      - transaction_status_in_progress = 0x00 , 正在进行中
      - transaction_status_committed = 0x01, 已提交
      - transaction_status_abborted = 0x02, 事务已回滚
      - transaction_status_sub_committed = 0x03, 子事务已提交
      
      事务id, 缩写为 kid, 是32 bit 的数字, 有以下3个特殊的
      
      - InvalidTransactionid = 0, 无效事务id
      - BootstrapTransactionid = 1, 系统表初始化事务id
      - frozenTransactionid = 2, 冻结事务id
      
      超过 2 的 31次方时, 会冻结旧的id, 所以比较时, 任意新事务id都会比冻结的大.
      
      ((int32)(id1 - id2)) < 0 , 返回真, 说明id1 比 id2 更早.
      
### 物理存储解构

    relation: 表或索引
    tuple: 表中的行, 即 row
    page: 磁盘中的数据块
    buffer: 内存中的数据块
    
#### 数据块结构
    
    块头 | 行指针1 | 行指针2 | 行指针3 | 行指针4 | 行指针5 ->
                       |
              |--------|              第5行 | 第4行
              |                        
    第3行 | 第2行 | 第一行 | 特殊数据
      

    数据块默认是 8kb, 最大32kb.  
    行指针向后顺序排列, 数据行向前相反方向排列, 之间是空闲空间.
    
    行指针: 32bit, 15bit 行内偏移, 2bit标记, 15bit内容长度,最大支持32kb数据；
    
    
#### tuple 解构

    header | value | value | value | value 
    
    header 包括以下:
      - oid , ctid, xmin, xmax ,cmin, cmax, ctid 
      - natts & infomask2 : 字段数
      - infomask : 当前行状态
      - hoff : 行头长度
      - bits : 数组, 标识行上哪些字段为空
      
#### 数据块空闲空间管理

    使用 "FSM" 的文件,记录每个数据块的空闲空间, free space map;
    
    8.4 版本后, 每一个表都有一个fsm 文件, 即 xxxoid_fsm
    
### 数据库控制文件  

    使用 pg_contraldata 显示控制文件内容.  
    
    database system identifier : 
      
      唯一标识一套数据库系统, 物理复制的主数据库和备数据库有相同的标识  
      
      ```
       select to_timestamp (((7088700542970782660 >> 32) & (2^32 -1) :: bigint));
      
      ```

    checkpoint :
      检查点只是一个数据库事件, 该事件触发后执行一系列操作;
      可以保证检查点前的脏数据都刷入磁盘中.
      
    standby 相关 :
      
      Minimum recovery ending lacation  主库 0/0 备库 0/xxxxxx
      Min recovery ending loc's timeline   主库 0 备库 1
      
### WAL

    write ahead log , 数据库重做日志
    
    LSN : log sequence number, 是一个不断增长的8字节(64bit)数字, 
      用于记录WAL 的绝对位置, 随着wal 增加, lsn也会不断增加.
      
    WAL 文件名的24个字符由3部分组成:  
      000000010000000000000001   
      ---8---|---8---|--------
      时间线    LogID   LogSeg
      
      LogID : LSN 的高32bit
      LogSeg : LSN的低32bit 再除以 WAL 文件的大小(一般64m)
      
    wal 循环写, 比append 少一次 i/o ;
    
      ```
      查看当前正在写的文件是哪一个
        select pg_walfile_name (pg_current_wal_lsn());
    
      查看wal
        pg_waldump
      ```
    
    pg 改名的实现,并不是更改文件名,
    实际上是创建一个新的硬链接文件指向旧文件, 然后删除旧文件.
    
### CommitLog 文件与事务ID
    
    pg_xact 目录下.
    
    有四种状态:  
        - transaction_status_in_progress = 0x00 , 正在进行中
        - transaction_status_committed = 0x01, 已提交
        - transaction_status_abborted = 0x02, 事务已回滚
        - transaction_status_sub_committed = 0x03, 子事务已提交

    commitLog 是一个位图文件, 需要2位表示事务状态.
    理论上最多2亿个事务, 所以CommitLog 最多占用512M.
    
    会被 VACUUM清理, autovacuum_freeze_max_age 默认2亿.
    
    事务 ID 是32 bit 的数字, 消耗完会有回卷问题,需要允许 vacuum, 
    新分配的ID 与老的之间距离还有1千万时会告警, 1百万时, 会宕机.
    
### 实例恢复

    导致数据库实例异常中止,通常有以下几个原因:  
      - 内存不足被OOM kill 或 用户 kill
      - 操作系统崩溃
      - 硬件故障导致机器停机或重启
      
    恢复后, replay 重做日志, 叫前滚；  
        取消未完成的事务, 叫回滚;  
        每次前滚时, 从哪个点开始, 这就是checkpoint 的概念.
        
    checkpoint_timeout 来控制checkpoint 的时间, 默认5分钟.
    
    为了保证重做时的"幂等", full_page_writes 为 on 时, 
    当数据块在前一次 checkpoint 后发生变化时, 会把数据块全部内容写入 Wal.
    恢复时, 先恢复整个数据块.
    
    未完成的数据不需要显式回滚, 因为在commitlog不是已提交状态,
    vacuum会清理掉.
    
### 热备份    
    
    备份过程:  
      调用函数 pg_start_backup() 开始热备  
      使用文件系统备份工具, 如 tar, cpio  
      调用函数结束热备份, 例如: select * from pg_start_backup()  
      把热备开始时的WAL 日志文件全部备份  
      
    恢复过程:  
      把备份的 WAL 文件复制到备份文件集中的 pg_wal 目录中
      启动数据库, 即可完成恢复
      
      
    原理:
      在执行函数 pg_start_backup() 时, 会做一次 checkpoint,  
      同时把 checkpoint 的点记录在特殊文件中, 即 backup_label文件  
      无论是用tar或者scp做文件系统备份时, 都会把 backup_label文件复制到备份中  
      在备份集中启动数据库,就会开始做实例恢复.  
      从指定的 checkpoint 开始应用WAL.
      
    这种备份只能启动一个, 不能对主库同时启动多个, 即独占型备份(exclusive backup)  
    另外 ,还有一个缺点, 在备份过程中如果主库宕机, 恢复时backup_label文件还存在,  
    这时, 分不清是备份库还是主库, 所以此时需要手工删除 backup_label文件.
    
    PG 9.1 之后 , pg_basebackup 工具提供了另一种备份方式.  
    称为 "非独占型备份" , pg_basebackup 使用一个连接到主库进行备份,没有并发  
    故, 9.6 之后, 把 "非独占型备份" 的功能以api 的方式暴露出来给大家用. 
    
    使用 " select pg_start_backup("xxxdb", false, false )" 可以启动, 
    结束时,也是以 "非独占型备份" 结束, 不会有 backup_label文件.
    
    热备份会强制设置 full_page_writes 为 on , 导致主库更多的WAL 日志.
    
    
    
### 其他黑科技

    index-only scan  
    heap-only tuples  
    
## pg 的特色功能

### 规则系统

    查询重写规则系统.  
    可以将发过来的SQL 命令在执行前, 通过内部规则定义改编为另一个sql 再执行
    
### 模式匹配和正则

    几种:
      - 传统sql 的like, ilike 以及 相应的操作符
      - sql99 标准新增的 similar to 操作符
      - posix 风格的正则
      - 模式匹配函数 substring
    
### listen & notify    

### 索引

#### 表达式索引

    支持通过函数创建索引, 插入时计算.
    
#### 部分索引

#### GiST  和 SP-GiST索引

    gist : 范围
    sp-gist : 空间分区索引
    
#### GIN 索引

    generalized inverted index , 广义倒排索引.
    
    对插入更新效率低, 最好先删除索引, 插入完再重建
    
#### BRIN 索引

    块范围索引.
    
### 序列    

    单独序列
    
### 咨询锁

    Advisory lock : 用一个64bit 的数字或者32bit的数字来表示, 
    并提供了一些函数进行加锁和释放的操作.
    
    9.1 后, 增加了事务级别的咨询锁, 事务结束后自动释放.
    
    例如:  
    pg_advisory_lock(key bigint)
    
    带 lock 的, 是加锁, 带 unlock 的是解锁;
    带 xact 的是事务级别的咨询锁, 
    含有 try 的,表示试图加锁, 无论是不是能获得锁, 立即返回; 
    获得返回true, 否则返回false;
    
    连接中断后, 锁会被释放.
    
    session 级别的锁和事务无关.
    
    
### SQL/MED

    SQL/MED 是SQL 语言中管理外部数据的一个扩展标准, 
    med 是 management of external data.
    
    9.1 版本开始支持.
    
    FDW, foreign data wrapper;  
    server : 外部数据库服务器, 相当于一个外部数据源, 需要指定外部数据源的FDW  
    User mapping : 用户映射, 把外部数据源的用户映射到本地用户, 用于权限控制.  
    Foreign Table : 外部表, 把外部数据源映射成数据库的一张表.  
    
### 全文检索

    默认英文
    
    ```
      select 'we love pg'::tsvector;
      # select 'we love pg'::tsvector;
          tsvector     
      ------------------
      'love' 'pg' 'we'
      (1 row)
      
    ```
    步骤:  
      创建 tsvector
      数据
      建立 GIN 索引    
      查询
      
    zahparser : 支持中文
    
### 并行查询

    11 版本后, 对并行查询能力大大增加
    
## 数据库优化

    指标: 响应时间 和 吞吐量
    
    优化工具:   
        - memtest86+, 内存测试工具
        - STREAM, 内存测试工具
        - sysbench, 综合测试, CPU, I/O, 数据库
        - pgbench, 自带测试工具, 可以仿真 TPC-B 的测试模型
        - fio
        - orion: oracle 的I/O测试工具
        
### 基本硬件


| 硬件             | 延迟      | 带宽      |
|:-----------------|:----------|:----------|
| Cache L1, L2, L3 | 0.5-15ns  | 20-60GB   |
| mem              | 30-100ns  | 2-12GB    |
| disk             | 5-20 ms   | 50-100m/s |
| ssd              | 10ns-1ms  | 50m-2gb/s |
| 网卡             | 100us-1ms | 10m-1gb/s |


    
### I/O 调优

####  打开 noatime

    ```
        /dev/sda1 / xfs noatime,errors=remount-ro 0 1
        
    ```

#### 调整预读

    ```
        blockdev --getra /dev/sdf
        blockdev --setra 4096 /dev/sdf
    
    ```

#### 调整虚拟内存参数
    
    swappiness, 0-100, 默认60 , 为0表示尽量使用物理内存, 取值越大越倾向于swap
    
    ```
        cat /proc/sys/vm/swapness
        
        更改 /etc/swappiness
        vm.swappiness = 0
        
    ```

    overcommit,   
      0, 启发式
      1, 总成功, 不管当前 malloc 内存情况
      2, 不允许 overcommit, 大于 swapd + N%mem,  vm.overcommit_ratio 控制比例
      
#### 写缓存优化

    - vm.dirty_background_ratio: 文件系统缓存脏页数量达到系统内存百分比时,触发刷盘, 异步刷盘
    - vm.dirty_ratio: 文件系统缓存脏页, 阻塞刷盘
    - vm.dirty_writeback_centisecs: 单位百分之一秒, 指定内核线程执行刷盘的间隔
    
#### 调整 I/O 调度器

    - cfq, 默认
    - deadline, 避免被饿死, 数据库比较适合
    - noop
    
### 性能监控
    
    各种性能视图


### 数据库配置优化

    
      
    
        
    
      
     
    
    
    
      
    
        
        
    

    
    


    
      
      
    

    
    
    
    
