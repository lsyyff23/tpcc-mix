##postgresql
[src-download](https://www.postgresql.org/ftp/source/v11.1/)
[1](https://blog.csdn.net/linuxpassion/article/details/52338064)
[2](https://blog.csdn.net/anzelin_ruc/article/details/8622770)
[3](http://www.pgsql.tech/doc_401_100)
### study
[1](https://www.tutorialspoint.com/postgresql/)

### postgresql 编译要求
    1 GNU make 3.8 +
    2 gcc  c89 以上
    3 gzip or bzip2 解压源码
    4 编译使用了GNU readline lib  --without-readline 可去除
      或者用libedit --with-libedit-preferred
      默认使用readline readlin-devel
      zlib 用于pg_dump pg_restore, --without-zlib 可移除


### postgresql数据类型

     Postgres                   |   MYSQL...
    ---
     boolean  1byte             | BOOL BOOLEAN
    ---
     smallint(2) int(4)         | tinyint(1) smallint(2) mediaint(3)
     bigint(8)                  | int(4) bigint(8)
     numeric(precision,scale)   | DECIMAL(M,N)
    ---
     varchar(n){max = 1G}       | varchar(n) {max ~~=~~ 64k}
     char(n)                    |
     text(~==~ MSYQL longtext)  |
    ---二进制
     bytea                      | blob longblob
    ---位串
     bit(n) bit varying(n)      | N/A
    ---
    date time timestamp(ms)     | timestamp(s) date  time
    ---
      ENUM                      | N/A
    ---几何
      point line lseg path      | N/A
      polygon cycle             |
    ---网络地址
       cidr inet macaddr        | N/A
    ---
        ARRAY                   | N/A
    ---
        复合== struct           | N/A
    ---
        XML                     | N/A
    ---
        JSON                    | N/A
    ---
        range                   | N/A
    ---对象标识符
        oid regproc regclass    | N/A
    ---伪类型
        any anyway anyelement   | N/A
        cstring internal record |
        language_handler trigger|
        void opaque             |
    ---其他
        UUID pg_lsn             | N/A

    ======================================
    类型转换
    1>
    testdb=# select 'col1', 'col2', 'col3';
     ?column? | ?column? | ?column? 
    ----------+----------+----------
     col1     | col2     | col3
    (1 row)

    testdb=# select smallint '1', int '2', bigint '3', bit '01010101';
     int2 | int4 | int8 |   bit    
    ------+------+------+----------
        1 |    2 |    3 | 01010101
    (1 row)


    2>
    testdb=# select CAST('5' as smallint), CAST('2' as int), CAST('3' as bigint), CAST('2018-09-29' AS DATE);
     int2 | int4 | int8 |    date    
    ------+------+------+------------
        5 |    2 |    3 | 2018-09-29
    (1 row)

    3>
    testdb=# select '5'::int, '2'::smallint, '6'::bigint, '2019-09-29'::date;
     int4 | int2 | int8 |    date
    ------+------+------+------------
        5 |    2 |    6 | 2019-09-29
    (1 row)

### postgres 数据库是进程架构模型

    Postmaster主进程
    查看pg启动的进程信息 select pid,usename,client_addr,client_port from pg_stat_activity;
    这些都是由postmaster fork出来的

    Syslogger（系统日志）进程
    postgresql.conf 的参数logging_collect on 时才打开，
    支持大小、文件名配置，log_directory log_filename等

    BgWriter（后台写）进程
    把共享内存中的脏页写入磁盘中，不能太快也不能太慢；
    配置参数以bgwriter_开头

    WalWriter（预写式日志写）进程
    WAL: Write Ahead Log -> xlog
    在修改数据之前，将修改操作记录到磁盘中，这样就不需要实时的把数据持久化到文件中了，
    即使宕机了，内存中的一些脏页未写入磁盘，在重启的时候通过执行WAL日志就可以恢复到宕机时的状态；
    pg_wal 目录下, 每个xlog文件大小默认是16M;

    PgArch（归档）进程 -- 不理解
    WAL日志会被覆盖，覆盖前PgArch会对其进行备份
    PITR技术：全量备份后，该技术将该时间点后WAL日志通过归档进行备份；

    AutoVacuum（自动清理）进程
    MVCC情况，在没有并发事务访问这些数据时，vaccum会清除那些旧数据；

    PgStat（统计数据收集）进程
    收集用于查询优化时的代价估算
    包括一个表和索引上的多少次插入、更新、删除操作，磁盘读写、行读取次数
    在pg_statistic 中


    共享内容
    提高读写性能；
    CLog、XLog、进程信息、锁信息、全局统计信息、子事务缓冲区、两阶段结构体、多事务缓冲区。。。

    本地内存:
    临时缓冲区  访问临时表的本地缓冲区
    work_mem  内部排序操作 和 hash表在使用临时磁盘文件之前使用的内存缓冲区
    maintenance_work_mem 在维护性操作中使用的内存缓冲区 vacuum, create index, alter table add foreign key


    alter USER postgres with password 'postgres';
    create user with password 'postgres'

### SQL语法

#### CREATE

    CREATE DATABASE testdb;
    CREATE TABLE disttab(col1 int, col2 int, col3 text) DISTRIBUTE BY HASH(col1);

#### delete

    [ WITH [ RECURSIVE ] with_query [, ...] ]
    DELETE FROM [ ONLY ] table_name [ * ] [ [ AS ] alias ]
        [ USING using_list ]
        [ WHERE condition | WHERE CURRENT OF cursor_name ]
        [ RETURNING * | output_expression [ [ AS ] output_name ] [, ...] ]

    1 WITH 语法
      [扩展]暂且不做支持

    2 FROM ONLY TABLE_NAME alias
      table_name  alias

    3 USING ?
      [扩展]暂且不支持
      USING  tablelist
             临时表？

    4 WHERE CONDITIONS
      CONDITIONS  == true 才执行
      : lhs = rhs
      : IN SELECT

    5 输出 返回 DELETE count
      [扩展]RETURN *

### psql-commands

#### \d

    资讯性
    (选项: S = 显示系统对象, + = 其余的详细信息)
    \d[S+]          列出表,视图和序列
    \d[S+]  名称      描述表，视图，序列，或索引
    \da[S]  [模式]    列出聚合函数
    \db[+]  [模式]     列出表空间
    \dc[S+] [PATTERN]      列表转换
    \dC[+]  [PATTERN]      列出类型强制转换
    \dd[S]  [PATTERN]      显示没有在别处显示的对象描述
    \ddp     [模式]    列出缺省权限
    \dD[S+] [PATTERN]      列出共同值域
    \det[+] [PATTERN]      列出引用表
    \des[+] [模式]    列出外部服务器
    \deu[+] [模式]     列出用户映射
    \dew[+] [模式]       列出外部数据封装器
     \df[antw][S+] [模式]    列出[只包括 聚合/常规/触发器/窗口]函数 
    \dF[+]  [模式]   列出文本搜索配置
    \dFd[+] [模式]     列出文本搜寻字典
    \dFp[+] [模式]     列出文本搜索解析器
    \dFt[+] [模式]   列出文本搜索模版
    \dg[+]  [PATTERN]      列出角色
    \di[S+] [模式]  列出索引
    \dl                   列出大对象， 功能与\lo_list相同
    \dL[S+] [PATTERN]      列出所有过程语言
    \dn[S+] [PATTERN]     列出所有模式
    \do[S]  [模式]   列出运算符
    \dO[S+] [PATTERN]      列出所有校对规则
    \dp     [模式]     列出表，视图和序列的访问权限
    \drds [模式1 [模式2]] 列出每个数据库的角色设置
    \ds[S+] [模式]    列出序列
    \dt[S+] [模式]     列出表
    \dT[S+] [模式]  列出数据类型
    \du[+]  [PATTERN]      列出角色
    \dv[S+] [模式]   列出视图
    \dE[S+] [PATTERN]      列出引用表
    \dx[+]  [PATTERN]      列出扩展
    \l[+]                列出所有的数据库
    \sf[+] FUNCNAME        显示函数定义
    \z      [模式]    和\dp的功能相同

#### \c

    \c testdb 连接数据库

#### 小结

    psql -l 查看数据库 ： psql -U postgres -d testdb -p 7001 -l
    有postgres template0 template1 ， 用户建库时默认是从template1 克隆的，拥有其函数和表
    psql -h <hostname or ip> -p <port> [db] [username]

    \d 查看表列表
    \d disttab  显示表结构定义
    \d t_index  索引信息
    \d *ttab   通配符*?
    匹配不同对象类型的\d
    \dt  table
    \di  index
    \ds  序列sequence
    \dv  view
    \df  function
    \dn  schema（模式） 列表  ~==~ 数据库的 database (概念来说，因为pg 不支持跨库，但支持跨模式)
        create schema osda;
        drop schema osda;
        alter schema osdba rename to osdbaold;
        alter schema osdba owner to web;
    \du  \dg 数据库中所有角色和用户
    \dT+ 查看自定义数据类型如枚举等

    \timing on|off 显示命令的执行时间
    \encoding utf8|gbk 客户端字符编码

    \pset 设置输出的样式
    \pset border 0  无边框
    \pset border 1  内边框
    \pset border 2  内外边框

    \x 每行的每列拆分成行
    \i 执行存储在外部文件的命令
        等价于 -f <filename> eg: psql -p 7001 testdb postgres -x -f selectdisttab.sql
        -x == \x ， -f == \i
    \echo  显示信息

    \g [file] or ; 输出到file
        eg: select * from disttab limit 10; \g re.txt

    \d [tab][tab] 2个table 补全命令

    \set AUTOCOMMIT ON|OFF  AUTOCOMMIT 大写
    \echo :AUTOCOMMIT
    \set ECHO_HIDDEN on|off 显示关闭某条命令实际执行的SQL

    -E 显示\d执行的内容
    psql -p 7001 testdb postgres -E

    psql postgresql://postgres:postgres@10.200.112.85:7001/postgres

#### transaction command

    get current transaction id:
    select txid_current();

#### 取消pid

    pg_cancel_backend(pid int)

### Two-phase commit

#### pg_prepared_xacts

    The view pg_prepared_xacts displays information about transactions that are currently prepared for two-phase commit (see PREPARE TRANSACTION for details).

[PREPARE TRANSACTION](https://www.postgres-xl.org/documentation/sql-prepare-transaction.html)

#### 两阶段提交：

    1 分布式情况中，事务会被全部执行或者回退；
    2 事务存在数据库中； 重启后仍可用；
    3 事务出现一台pg失败，则全部失败回退；
    ---- Q1 能否获取处理结果！？？ 这样有助于处理问题；
     Q2 占用内存的情况？

命令流程如下

    begin;
    insert into ...
    prepare transaction 'tid1';
    commit prepared 'tid1';

#### pg_prepared_xact view

    CREATE OR REPLACE VIEW pg_prepared_xacts AS
        SELECT P.transaction, P.gid, P.prepared,
               U.rolname AS owner, D.datname AS database
        FROM pg_prepared_xact() AS P
        (transaction , gid , prepared , ownerid , dbid )
             LEFT JOIN pg_authid U ON P.ownerid = U.oid
             LEFT JOIN pg_database D ON P.dbid = D.oid
        where P.gid = '15_140672837338880_1538992711671_52_263196';

#### txn commit error
commit error

    commit 前，dn1_master failed
    WARNING:  unexpected EOF on datanode oid connection: 16385
    WARNING:  Error while PREPARING transaction _$XC$164056:cd1:F:2:0:-560021589:352366662 on node dn2. Administrative action may be required to abort this transaction on the node
    ERROR:  terminating connection due to administrator command
    testdb=# commit
    ;
    WARNING:  there is no transaction in progress
    COMMIT

    ERROR:  Failed to receive more data from data node 16385
    ERROR:  Failed to get pooled connections
    ERROR:  terminating connection due to unexpected postmaster exit

    ERROR:  terminating connection due to administrator command

[postgresql事务提交失败导致锁表的解决办法](https://blog.csdn.net/benxiaojun/article/details/44855311)


### pg_ctl

    [postgres@dbca ~]$ /usr/local/postgresql/bin/pg_ctl --help
    pg_ctl is a utility to initialize, start, stop, or control a PostgreSQL server.

    Usage:
      pg_ctl init[db] [-D DATADIR] [-s] [-o OPTIONS]
      pg_ctl start    [-D DATADIR] [-l FILENAME] [-W] [-t SECS] [-s]
                      [-o OPTIONS] [-p PATH] [-c]
      pg_ctl stop     [-D DATADIR] [-m SHUTDOWN-MODE] [-W] [-t SECS] [-s]
      pg_ctl restart  [-D DATADIR] [-m SHUTDOWN-MODE] [-W] [-t SECS] [-s]
                      [-o OPTIONS] [-c]
      pg_ctl reload   [-D DATADIR] [-s]
      pg_ctl status   [-D DATADIR]
      pg_ctl promote  [-D DATADIR] [-W] [-t SECS] [-s]
      pg_ctl kill     SIGNALNAME PID

    Common options:
      -D, --pgdata=DATADIR   location of the database storage area
      -s, --silent           only print errors, no informational messages
      -t, --timeout=SECS     seconds to wait when using -w option
      -V, --version          output version information, then exit
      -w, --wait             wait until operation completes (default)
      -W, --no-wait          do not wait until operation completes
      -?, --help             show this help, then exit
    If the -D option is omitted, the environment variable PGDATA is used.

    Options for start or restart:
      -c, --core-files       allow postgres to produce core files
      -l, --log=FILENAME     write (or append) server log to FILENAME
      -o, --options=OPTIONS  command line options to pass to postgres
                             (PostgreSQL server executable) or initdb
      -p PATH-TO-POSTGRES    normally not necessary

    Options for stop or restart:
      -m, --mode=MODE        MODE can be "smart", "fast", or "immediate"

    Shutdown modes are:
      smart       quit after all clients have disconnected
      fast        quit directly, with proper shutdown (default)
      immediate   quit without complete shutdown; will lead to recovery on restart

    Allowed signal names for kill:
      ABRT HUP INT KILL QUIT TERM USR1 USR2

    Report bugs to <pgsql-bugs@postgresql.org>.

#### shutdown pg process

Postgres异常关闭重启后会进行实例恢复
shutdown-mode有如下几种模式：

    1. smart: 等所有的连接中止后，关闭数据库。如果客户端连接不终止，则无法关闭数据库。
    2. fast: 快速关闭数据库， 断开客户端的连接，让已有的事务回滚，然后正常关闭数据库。
    3. immediate: 立即关闭数据库，立即停止数据库进程，直接退出，下次启动时会进行实例恢复。

    pg_ctl stop -m immediate -D /home/postgres/pgdata/dn1_master


### others
execRemote.c
执行 pgxc_node_remote_prepare();

[system information functions](https://www.postgresql.org/docs/10/static/functions-info.html)


### psql-comands

    -\x 展开显示开关
    - show all; 查看所有配置，列出来的配置项都可以 show [config]; 来单独显示
          如：testdb=# show autovacuum_vacuum_scale_factor;
              autovacuum_vacuum_scale_factor 
              --------------------------------
               0.2
      其实显示的是pg_catalog.pg_settings表的数据内容

    --设置autocommit 及显示
    testdb=# \set autocommit on
    testdb=# \echo autocommit
    autocommit
    testdb=# \echo :autocommit
    on
    testdb=# \set autocommit off
    testdb=# \echo :autocommit
    off

SQL 命令分为 DQL DML DDL

    DQL -> SELECT  查询
    DML -> data mannipulation language  INSERT UPDATE DELETE 更新
    DDL -> Data definition language ,  CREATE DROP MODIFY INDEX tables  定义数据库对象（库表）
    truncate table disttab; 属于DDL 重新定义一张新表把原表丢弃；


### test table

    CREATE TABLE disttab(col1 int, col2 int, col3 text) DISTRIBUTE BY HASH(col1);
    CREATE OR REPLACE VIEW disttab_view AS select col1, col2 col3 from disttab;

    create database testdb;
    \c testdb;
    CREATE TABLE test2(id text, col1 int, col2 int, col3 text, primary key(id)) DISTRIBUTE BY HASH(id);
    alter table test2 rename to test2_real;
    CREATE OR REPLACE VIEW test2 AS SELECT id, col1, col2, col3 FROM test2_real;
    alter table test2_real add column utime bigint default 0;
    alter table test2_real add column ucount bigint default 0;


### using
    insert into disttab values (1, 1, '1'), (2, 2, '2'), (11, 11, '11'), (22, 22, '22');

    execute direct on (dn2) 'select * from disttab;';
    execute direct on (dn1) 'select * from disttab;';

    prepare transaction '1';
    commit prepared '1';

    /usr/local/pgxl/bin/initdb --nodename dn5 -D /usr/local/pgdata/dn5_master -E UTF8 --locale=C -U postgres

    psql -U postgres -p 7001 -d testdb

    begin;
    insert into disttab values (1, 1, '1'), (2, 2, '2'), (11, 11, '11'), (22, 22, '22');
    prepare transaction 'AAAA_1';
    commit prepared 'AAAA_1';
    execute direct on (cd1) 'select * from pg_prepared_xacts;';
    execute direct on (dn1) 'select * from pg_prepared_xacts;';
    execute direct on (dn2) 'select * from pg_prepared_xacts;';

    commit prepared 'AAAA_1';

    execute direct on (cd1) 'select * from pg_prepared_xacts;';
    execute direct on (dn1) 'select * from pg_prepared_xacts;';
    execute direct on (dn2) 'select * from pg_prepared_xacts;';

    execute direct on (cd1) 'select pgxc_is_committed(''68340'');';

    set xc_maintenance_mode to on;
    execute direct on (dn1) 'commit prepared ''39_140298344699648_1542244130750_31_68340'';';
    execute direct on (dn2) 'commit prepared ''39_140298344699648_1542244130750_31_68340'';';

    execute direct on (cd1) 'commit prepared ''39_140298344699648_1542244130750_31_68340'';';

    execute direct on (dn1) 'select pgxc_is_committed(''68311'');';

    execute direct on (dn1) 'rollback prepared ''33_140298478917376_1542244130493_33_68311'';';
    =================
    pg_prepared_xacts 
    1 
    68340 | 39_140298344699648_1542244130750_31_68340 | 2018-11-15 09:08:50.552306+08 | postgres | testdb
    在dn1 无， dn2 无, cd1 有， 
    pgxc_is_committed(), dn1 t, dn2 t, cd  null
    执行maintenance commit/rollback cd1 时 数据都写入成功!!!并增加了数据

    2 做cd2 可以执行cd1在dn1 dn2做的数据， 并完成命令
    =============

    psql postgresql://postgres:postgres@10.200.110.162:7001/postgres
    psql postgresql://postgres:postgres@10.200.110.162:7001/postgres -c "alter USER postgres with password 'postgres';"


### chapter 13

warm standby server: sync data from master or applications, and no provide reading
hot standby server: sync data from master or applications, and provide reading!

### postgresql 编译安装

    yum install -y readline readline-devel

    [cnblogs-zhoulf]https://www.cnblogs.com/zhoulf/p/4040768.html
    ./configure --prefix=/usr/local/postgresql
    ./configure --prefix=/usr/local/pgxl

    make -j24; make install

### postgresql using

    #切换用户
    groupadd -r postgres;
    useradd -g postgres -s /sbin/nologin postgres;

    su postgres
psql postgresql://postgres:postgres@10.200.110.162:7001/ -c "alter USER postgres with password 'postgres';"

    #初始化数据库
    mkdir /usr/local/pgdata -p;
    chown -R postgres:postgres /usr/local/postgresql /usr/local/pgdata;
    initdb -D /usr/local/pgdata

    su - postgres -c "/usr/local/postgresql/bin/initdb -D /usr/local/pgdata"
    #启动服务
    su - postgres -c "/usr/local/postgresql/bin/pg_ctl -D /usr/local/pgdata -l /usr/local/pgdata/logfile start"

    #停止服务
    pg_ctl -D /usr/local/pgdata -l /usr/local/pgdata/logfile stop

    #--------------------允许远程连接---------------------------
    #修改客户端认证配置文件pg_hba.conf，将需要远程访问数据库的IP地址或地址段加入该文件
    vi /var/postgresql/data/pg_hba.conf

    #在文件的最下方加上下面的这句话
    host    all         all         0.0.0.0/0             trust

    #设置监听整个网络，查找“listen_addresses ”字符串，
    vi /var/postgresql/data/postgresql.conf

    #修改为如下：
    listen_addresses = '*'


    #端口是否启用
    netstat -anp | grep 5432
    select pg_reload_conf();


### postgresql 参数优化

http://blog.163.com/digoal@126/blog/static/16387704020121952051174/

[Postgres XL压力测试与性能调优](https://www.wanghengbin.com/2016/07/04/postgres-xl-presstest-optimize/)
    important !!!

[taobao](http://mysql.taobao.org/monthly/2016/02/07/)

    shared_buffers = 36GB

    work_mem = 256MB
    maintenance_work_mem = 1GB
    autovacuum_work_mem = 1GB
    dynamic_shared_memory_type = posix

    bgwriter_delay = 10ms
    bgwriter_lru_maxpages = 1000
    bgwriter_lru_multiplier = 10.0
    wal_sync_method = open_sync
    full_page_writes = off
    synchronous_commit = off
    wal_buffers = 1GB
    wal_writer_delay = 10ms
    effective_cache_size = 40GB

    #huge_pages = try
    #wal_level = minimal
说明

    max_connections = 300       # (change requires restart)
    unix_socket_directories = '.'   # comma-separated list of directories

    shared_buffers = 194GB       # 尽量用数据库管理内存，减少双重缓存，提高使用效率
    huge_pages = on           # on, off, or try  ，使用大页
    work_mem = 256MB # min 64kB  ， 减少外部文件排序的可能，提高效率
    maintenance_work_mem = 2GB  # min 1MB  ， 加速建立索引

    autovacuum_work_mem = 2GB   # min 1MB, or -1 to use maintenance_work_mem  ， 加速垃圾回收

    dynamic_shared_memory_type = mmap      # the default is the first option
    vacuum_cost_delay = 0      # 0-100 milliseconds   ， 垃圾回收不妥协，极限压力下，减少膨胀可能性
    bgwriter_delay = 10ms       # 10-10000ms between rounds    ， 刷shared buffer脏页的进程调度间隔，尽量高频调度，减少用户进程申请不到内存而需要主动刷脏页的可能（导致RT升高）。
    bgwriter_lru_maxpages = 1000   # 0-1000 max buffers written/round ,  一次最多刷多少脏页
    bgwriter_lru_multiplier = 10.0          # 0-10.0 multipler on buffers scanned/round  一次扫描多少个块，上次刷出脏页数量的倍数
    effective_io_concurrency = 2           # 1-1000; 0 disables prefetching ， 执行节点为bitmap heap scan时，预读的块数。从而
    wal_level = minimal         # minimal, archive, hot_standby, or logical ， 如果现实环境，建议开启归档。
    synchronous_commit = off    # synchronization level;    ， 异步提交
    wal_sync_method = open_sync    # the default is the first option  ， 因为没有standby，所以写xlog选择一个支持O_DIRECT的fsync方法。
    full_page_writes = off      # recover from partial page writes  ， 生产中，如果有增量备份和归档，可以关闭，提高性能。
    wal_buffers = 1GB           # min 32kB, -1 sets based on shared_buffers  ，wal buffer大小，如果大量写wal buffer等待，则可以加大。
    wal_writer_delay = 10ms         # 1-10000 milliseconds  wal buffer调度间隔，和bg writer delay类似。
    commit_delay = 20           # range 0-100000, in microseconds  ，分组提交的等待时间
    commit_siblings = 9        # range 1-1000  , 有多少个事务同时进入提交阶段时，就触发分组提交。
    checkpoint_timeout = 55min  # range 30s-1h  时间控制的检查点间隔。
    max_wal_size = 320GB    #   2个检查点之间最多允许产生多少个XLOG文件
    checkpoint_completion_target = 0.99     # checkpoint target duration, 0.0 - 1.0  ，平滑调度间隔，假设上一个检查点到现在这个检查点之间产生了100个XLOG，则这次检查点需要在产生100*checkpoint_completion_target个XLOG文件的过程中完成。PG会根据这些值来调度平滑检查点。
    random_page_cost = 1.0     # same scale as above  , 离散扫描的成本因子，本例使用的SSD IO能力足够好
    effective_cache_size = 240GB  # 可用的OS CACHE
    log_destination = 'csvlog'  # Valid values are combinations of
    logging_collector = on          # Enable capturing of stderr and csvlog
    log_truncate_on_rotation = on           # If on, an existing log file with the
    update_process_title = off
    track_activities = off
    autovacuum = on    # Enable autovacuum subprocess?  'on'
    autovacuum_max_workers = 4 # max number of autovacuum subprocesses    ，允许同时有多少个垃圾回收工作进程。
    autovacuum_naptime = 6s  # time between autovacuum runs   ， 自动垃圾回收探测进程的唤醒间隔
    autovacuum_vacuum_cost_delay = 0    # default vacuum cost delay for  ， 垃圾回收不妥协

### pg -benchmark
    https://segmentfault.com/a/1190000007012082


### PGXZ
    [pgxz](https://mp.weixin.qq.com/s?__biz=MzA3MDQ4MzQzMg==&mid=2665690408&idx=2&sn=60ff60ffb588e66b8965f74ee806475c&scene=2&srcid=0607JdTatK8eNFCow3Cn5QHN&from=timeline&isappinstalled=0#wechat_redirect)

    [其他](https://zhuanlan.zhihu.com/p/36670097)

### PGTune
    https://pgtune.leopard.in.ua/#/

### 内核参数

    SHMMAX  Maximum size of shared memory segment (bytes)   at least 1kB, but the default is usually much higher
    SHMMIN  Minimum size of shared memory segment (bytes)   1
    SHMALL  Total amount of shared memory available (bytes or pages)    same as SHMMAX if bytes, or ceil(SHMMAX/PAGE_SIZE) if pages, plus room for other applications
    SHMSEG  Maximum number of shared memory segments per process    only 1 segment is needed, but the default is much higher
    SHMMNI  Maximum number of shared memory segments system-wide    like SHMSEG plus room for other applications
    SEMMNI  Maximum number of semaphore identifiers (i.e., sets)    at least ceil((max_connections + autovacuum_max_workers + max_worker_processes + 5) / 16) plus room for other applications
    SEMMNS  Maximum number of semaphores system-wide    ceil((max_connections + autovacuum_max_workers + max_worker_processes + 5) / 16) * 17 plus room for other applications
    SEMMSL  Maximum number of semaphores per set    at least 17
    SEMMAP  Number of entries in semaphore map  see text
    SEMVMX  Maximum value of semaphore  at least 1000 (The default is often 32767; do not change unless necessary)

[rf](https://blog.csdn.net/silence_ljh/article/details/24479223)
/etc/sysctl.conf

SHM 和 SEM

    vm.overcommit_memory = 1
    net.ipv4.tcp_max_syn_backlog = 4100
    vm.max_map_count = 262144
    net.ipv4.ip_local_port_range =  5000 65000
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_fin_timeout = 30
    net.core.somaxconn = 2048
    net.ipv4.tcp_low_latency = 0
    kernel.sem = 50100 128256000 50100 2560
    vm.dirty_bytes=33554432

    net.ipv4.tcp_fin_timeout = 15
    net.ipv4.tcp_tw_recycle = 1

    vm.nr_hugepages = 0

大页计算
 [Linux Huge Pages](https://www.postgres-xl.org/documentation/kernel-resources.html)
$ head -1 $PGDATA/postmaster.pid
4170
$ grep ^VmPeak /proc/4170/status
VmPeak:  6490428 kB
$ grep ^Hugepagesize /proc/meminfo
Hugepagesize:       2048 kB

sysctl -w vm.nr_hugepages=6490428/2048

### configures

[19.19. Postgres-XL Specific Parameters](https://www.postgres-xl.org/documentation/pg-xc-specifics.html)
[huge_pages=try](https://www.postgres-xl.org/documentation/kernel-resources.html#LINUX-HUGE-PAGES)

#### pgxl configure settings

    listen_addresses='*'
    persistent_datanode_connections=on
    max_connections=500
    shared_buffers=12GB
    huge_pages=try
    max_prepared_transactions=1000
    work_mem=36MB
    maintenance_work_mem=512MB
    dynamic_shared_memory_type=posix
    autovacuum=off
    bgwriter_delay=10000ms
    bgwriter_flush_after=2MB
    effective_io_concurrency=16
    max_worker_processes=16
    max_parallel_workers_per_gather=4
    max_parallel_workers=16
    wal_level=replica
    fsync=off
    synchronous_commit=off
    wal_sync_method=fdatasync
    full_page_writes=off
    wal_buffers=32MB
    wal_writer_delay=10000ms
    wal_writer_flush_after=32MB
    commit_delay=10000ms
    commit_siblings=100
    checkpoint_timeout=30min
    checkpoint_flush_after=2MB
    checkpoint_warning=0
    max_wal_size=2GB
    min_wal_size=256MB
    max_wal_senders=0
    hot_standby=off
    effective_cache_size=24GB
    logging_collector=on
    log_truncate_on_rotation=on
    log_min_messages=info
    track_activities=off
    track_counts=off
    autovacuum=false
    shared_queues=128

##### cd:

    listen_addresses='*'
    persistent_datanode_connections=on
    max_connections=500
    shared_buffers=12GB
    huge_pages=try
    max_prepared_transactions=1000
    work_mem=36MB
    maintenance_work_mem=512MB
    dynamic_shared_memory_type=posix
    autovacuum=off
    bgwriter_delay=10000ms
    bgwriter_flush_after=2MB
    effective_io_concurrency=16
    max_worker_processes=16
    max_parallel_workers_per_gather=4
    max_parallel_workers=16
    wal_level=replica
    fsync=off
    synchronous_commit=off
    wal_sync_method=fdatasync
    full_page_writes=off
    wal_buffers=32MB
    wal_writer_delay=10000ms
    wal_writer_flush_after=32MB
    commit_delay=10000ms
    commit_siblings=100
    checkpoint_timeout=30min
    checkpoint_flush_after=2MB
    checkpoint_warning=0
    max_wal_size=2GB
    min_wal_size=256MB
    max_wal_senders=0
    hot_standby=off
    effective_cache_size=24GB
    logging_collector=on
    log_truncate_on_rotation=on
    log_min_messages=info
    track_activities=off
    track_counts=off
    autovacuum=false
    shared_queues=128

    checkpoint_completion_target = 0.9
    default_statistics_target = 100
    random_page_cost=4
    bgwriter_lru_maxpages=1000
    bgwriter_lru_multiplier=10.0
    maintenance_work_mem=1GB
    autovacuum_work_mem=1GB
    fsync:off

    update_process_title=off
    old_snapshot_threshold=30s
    autovacuum_naptime=20min


### pgbench
https://segmentfault.com/a/1190000007012082

[help](https://blog.csdn.net/sunziyue/article/details/50997867)
http://valleylord.github.io/post/201411-postgres-pgbench/

pgbench –help

    初始化选项：
    -i 调用初始化模式
    -F NUM 填充因子
    -s scalor 规模（与产生数据量大小有关）

    Benchmarking选项：
    -c NUM  数据库客户端并发数（默认：1）
    -C （为每个事务建立新的连接）
    -D VARNAME=VALUE 通过客户脚本为用户定义变量
    -f FILENAME 从文件FILENAME读取事务脚本
    -j NUM  线程数（默认：1）
    -i  写事务时间到日志文件
    -M {simple|extended|prepared} 给服务器提交查询的协议
    -n 在测试之前不运行VACUUM
    -N 不更新表“pgbench_tellers” “pgbench_branches”
    -r 报告每条命令的平均延迟
    -s NUM 在输出中报告规模因子
    -S 执行 SELECT-only事务
    -t NUM 每个客户端运行的事务数（默认：10）
    -T NUM benchmark测试时间（单位：秒）
    -v 在测试前清空所有的四个标准表

    常用选项：
    -d 输出打印调试信息
    -h HOSTNAME 数据库服务器主机或socket 目录
    -U USERNAME 指定数据库用户的连接
    --help 显示帮助信息，然后退出
    --version 输出版本信息，然后退出

    At the default “scale factor” of 1, the tables initially contain this many rows:
    table                   # of rows
    ---------------------------------
    pgbench_branches        1
    pgbench_tellers         10
    pgbench_accounts        100000
    pgbench_history         0

test

    1
    psql postgresql://postgres:postgres@10.200.110.162:7001/ -c "alter USER postgres with password 'postgres';"
    psql postgresql://postgres:postgres@10.200.110.162:7001/ -c "create database pgbenchsql;"
    2
    #10GB dataset
    #./pgbench -i -F 100 -s 714 -h 127.0.0.1 -p 6432 -U postgres benchmarksql
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -i -F 100 -s 714


    #20GB dataset
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -i -F 100 -s 1428

    #80GB dataset
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -i -F 100 -s 5712

    #160GB dataset
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -i -F 100 -s 11424


    查看数据库大小

    psql postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c "select pg_database_size('pgbenchsql')/1024/1024/1024|| 'GB';"
    or
    psql postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c "select pg_size_pretty(pg_database_size('pgbenchsql'));"

    3
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c 10 -j 10 -s 714 -r -T 1800
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c 20 -j 20 -M prepared -n -s 1428 -T 1800 -r
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c 50 -j 50 -M prepared -n -s 1428 -T 1800 -r


example1:

    /usr/local/pgxl/bin/pgbench postgresql://postgres:postgres@10.200.110.162:30001/pgbenchsql -i -F 100 -s 500
    psql postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c "select pg_size_pretty(pg_database_size('pgbenchsql'));"
    /usr/local/pgxl/bin/pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c 20 -j 20 -s 500 -r -T 900
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -c 20 -j 20 -s 500 -S -r -T 180 -v

example2:

    pgbench postgresql://postgres:postgres@10.200.110.162:5432/pgbenchsql -i -F 100 -s 1000
    psql postgresql://postgres:postgres@10.200.110.162:5432/pgbenchsql -c "select pg_size_pretty(pg_database_size('pgbenchsql'));" ;\
    pgbench postgresql://postgres:postgres@10.200.110.162:5432/pgbenchsql -c 20 -j 20 -s 1000 -r -T 180;\
    pgbench postgresql://postgres:postgres@10.200.110.162:5432/pgbenchsql -c 20 -j 20 -s 1000 -S -r -T 180 -v;

example3:

    psql postgresql://postgres:postgres@10.200.110.162:7001/ -c "alter USER postgres with password 'postgres';"
    psql postgresql://postgres:postgres@10.200.110.162:7001/ -c "create database pgbenchsql;"
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -i -F 100 -s 500;

    单条脚本测试
    pgbench postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -n  -f t1_select.sql  -t 1 -l -s 500

    psql postgresql://postgres:postgres@mq:30001/ -c "create database pgbenchsql;"
    pgbench postgresql://postgres:postgres@mq:30001/pgbenchsql -i -F 100 -s 10
    pgbench postgresql://postgres:postgres@mq:30001/pgbenchsql -c 1 -s 10 -r -T 180;

    execute direct on (cd1) 'select count(*) from pgbenchsql.pgbench_tellers';
    execute direct on (dn1) 'select count(*) from pgbenchsql.pgbench_tellers';
    execute direct on (dn2) 'select count(*) from pgbenchsql.pgbench_tellers';

### pg_dump
    pg_dump -U username -W -s -t tablename

    -s, --schema-only            只转储模式, 不包括数据
    -t, --table=TABLE            只转储指定名称的表
    pg_dump postgresql://postgres:postgres@10.200.110.162:7001/pgbenchsql -W -s -t pgbench_accounts

### coordinator 代码解析

#### Select 过程
    根据pstack 抓取的

    PostmasterMain
        ->ServerLoop
            ->BackendStartup
                ->BackendRun
                    ->PostgresMain
                        ->exec_simple_query { //postgres.c
                            pg_parse_query()
                            PortalDefineQuery
                        *   ->PortalStart():  //pquery.c
                                ->GetTransactionSnapshot
                                    ->GetPGXCSnapshotData
                                        ->GetSnapshotDataFromGTM
                                            ->GetSnapshotGTM
                                                ->get_snapshot
                                                    ->gtmpqWaitTimed
                                                        ->gtmpqSocketCheck
                                                            ->gtmpqSocketPoll
                                                                ->__poll_nocancel
                                ->ExecutorStart //execMain.c
                                    ->standard_ExecutorStart
                                        ->InitPlan
                                            ->ExecInitNode
                        *   ->PortalRun: ->PortalRunSelect 如下
                                ->PortalRunSelect
                                    ->ExecutorRun
                                        ->standard_ExecutorRun
                                            ->ExecutePlan
                                                ->ExecProcNode
                                                    ->ExecRemoteQuery
                                                        ->FetchTuple
                                                            ->pgxc_node_receive
                                                                ->__poll_nocancel
                        *   PortalDrop
                            }



### 性能调优工具大全
[性能调优工具大全](http://www.pgsql.tech/article_101_10000040)
