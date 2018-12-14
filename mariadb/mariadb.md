### MariaDB

#### 编译安装

##### 1 删系统mariadb libs

    - 删配置
        查找：
            find -H /etc/ | grep my.c
        删除：
            rm -rf /etc/my.cnf /etc/my.cnf.d/
    - 删除libs
        查找
            rpm -qa|grep mariadb-libs
        删除
            rpm -e mariadb-libs-5.5.52-1.el7.x86_64 --nodeps
##### 2 下载
        wget https://codeload.github.com/MariaDB/server/tar.gz/mariadb-10.3.11
        mv mariadb-10.3.11 mariadb-10.3.11.tar.gz
        tar -zxvf mariadb-10.3.11.tar.gz

##### 3 安装依赖库
        yum -y install boost \
        yum -y install boost-devel \
        yum -y install libaio; \
        yum -y install libaio-devel;  \
        yum -y install bison;\
        yum -y install bison-devel; \
        yum -y install zlib-devel ; \
        yum -y install openssl; \
        yum -y install openssl-devel ; \
        yum -y install ncurses ; \
        yum -y install ncurses-devel; \
        yum -y install libcurl-devel; \
        yum -y install libarchive-devel ; \
        yum -y install lsof ; \
        yum -y install wget; \
        yum -y install gcc ; \
        yum -y install gcc-c++; \
        yum -y install make; \
        yum -y install cmake; \
        yum -y install perl; \
        yum -y install kernel-headers; \
        yum -y install kernel-devel ; \
        yum -y install pcre-devel;


##### 4 编译安装cmake
       # -DWITHOUT_TOKUDB=1 \ usr/local/yealink/mysqlconf
         -DWITH_TOKUDB=1 \

        cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/yealink/mariadb \
         -DMYSQL_DATADIR=/usr/local/yealink/mariadbdata \
         -DSYSCONFDIR=/etc \
         -DWITH_INNOBASE_STORAGE_ENGINE=1 \
         -DWITH_ARCHIVE_STPRAGE_ENGINE=1 \
         -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
         -DWIYH_READLINE=1 \
         -DWIYH_SSL=system \
         -DVITH_ZLIB=system \
         -DWITH_LOBWRAP=0 \
         -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
         -DDEFAULT_CHARSET=utf8 \
         -DDEFAULT_COLLATION=utf8_general_ci

        make -j12; make install

##### 5 添加权限：
        groupadd -r mysql;
        useradd -g mysql -s /sbin/nologin mysql;
        mkdir -p /usr/local/yealink/mariadb;
        mkdir -p /usr/local/yealink/mariadbdata;
        chown -R mysql:mysql /usr/local/yealink/mariadb /usr/local/yealink/mariadbdata;

##### 配置MariaDB
        cd /usr/local/yealink/mariadb;
        cp support-files/wsrep.cnf /etc/my.cnf;
        scripts/mysql_install_db --user=mysql --datadir=/usr/local/yealink/mariadbdata;
        #cp support-files/my-large.cnf /usr/local/yealink/mysqlconf/my.cnf;
        cp support-files/mysql.server /etc/rc.d/init.d/mysqld;

##### 启停
        /etc/rc.d/init.d/mysqld start

##### 修改密码
        './bin/mysqladmin' -u root password 'new-password'
        './bin/mysqladmin' -u root -h localhost.localdomain password 'new-password'
        bin/mysql_secure_installation

##### 支持root用户允许远程连接mysql数据库
        CREATE USER 'linkbench'@'%' IDENTIFIED BY 'mysql';
        grant all privileges on *.* to 'root'@'%' identified by "mysql" with grant option;
        flush privileges;

##### 参考

    1.https://www.cnblogs.com/freeweb/p/5991374.html
    2.https://segmentfault.com/a/1190000009909776
    3.tokudb 使用说明：
        https://www.cnblogs.com/tianshifu/p/8410822.html
    4.https://blog.csdn.net/qq_32828933/article/details/82720018

------
#### 参数优化

##### 配置文件

    /etc/my.cnf

###### 参数解读

        https://www.centos.bz/2018/02/mariadb-mysql%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6my-cnf%E8%A7%A3%E8%AF%BB/

###### 参考

    - mariadb配置文件优化参数

        https://linux.cn/article-9325-1.html

    - 优化 MySQL： 3 个简单的小调整

        https://www.jianshu.com/p/219e45e7b050

    - 15 个有用的 MySQL/MariaDB 性能调整和优化技巧

        https://linux.cn/article-5730-1.html

###### 记录

参数调优内容：

1. 内存利用方面
2. 日志控制方面
3. 文件IO分配，空间占用方面
4. 其它相关参数

具体如下：

1. mysql内存大小：

        innodb_buffer_pool_size     设为内存的70%-80%

2. 提高写日志的性能

        （1）innodb_flush_log_at_trx_commit 配置设定为0；减少写日志的频率。
            将 innodb_flush_log_at_trx_commit 配置设定为0；按过往经验设定为0，插入速度会有很大提高。默认为1 

            0: Write the log buffer to the log file（刷到内存的log_file中）and flush the log file （刷到硬盘）every second, but do nothing at transaction commit. 
            1：the log buffer is written out to the log file at each transaction commit and the flush to disk operation is performed on the log file 
            2：the log buffer is written out to the file at each commit, but the flush to disk operation is not performed on it （仍然每秒刷硬盘一次）
        （2）innodb_log_buffer_size 增大。 增大日志缓存，减少写数据文件次数。
            设置为一秒的日志大小即可
            这个参数设置 InnoDB 来往磁盘上的日志文件写操作的缓冲区的大小。通过内存缓冲来延缓磁盘 I/O
            以提高访问的效率。 因为 MySQL 每秒都会将日志缓冲区的内容刷新到日志文件，因此无需设置超过
             1 秒所需的内存空间。通常设置为 8 ～ 16MB 就足够了，默认值是 1MB 
        （3）innodb_log_file_size 增大。
            参考自：计算日志大小
            默认日志大小。增大日志文件大小，可以减少数据库checkpoint操作。
            大小必须小于4G,不过设置多个，innodb_log_files_in_group  =2 ，默认是2。
            估算大小的原则；innodb_log_file_size×2大小等于一个小时的数据写入量就可以了。也就是说一个小时触发一次checkpoint。这是一个经验值，当然越大越好，但是很大的情况不一定带来性能大的提升。
            估算大小的方法：
                mysql> pager grep sequence
                PAGER set to 'grep sequence'
                mysql> show engine innodb status\G select sleep(60); show engine innodb status\GLog sequence number 84 3836410803
                1 row in set (0.06 sec)
                1 row in set (1 min 0.00 sec)
                Log sequence number 84 3838334638
                1 row in set (0.05 sec)
                mysql> pager
                Default pager wasn't set, using stdout.

                mysql>select(3838334638-3836410803)/1024/1024asMB_per_min;

                +------------+

                |MB_per_min|

                +------------+

                |1.83471203|

                +------------+
                然后拿这个值MB_per_min×60/2，就是innodb_log_file_size合理值。
                如果修改了innodb_log_file_size，关闭数据库后，要把旧的ib_logfile移走，再启动数据库，否则会崩溃.
                参考：修改MYSQL的innodb_log_file_size导致的MYSQL崩溃 
        （4）sync_binlog =  N  提高日志写入磁盘速率  可设置为很大，比如2000，看具体场景
            N>0  — 每向二进制日志文件写入N条SQL或N个事务后，则把二进制日志文件的数据刷新到磁盘上；
            N=0  — 不主动刷新二进制日志文件的数据到磁盘上，而是由操作系统决定；

3. 提高写数据的性能

    （1）将 innodb_autoextend_increment 增大。减少表空间扩展次数

4. 其他可调参数

        (1) innodb_flush_method             = O_DIRECT
        innodb_flush_method这个参数控制着innodb数据文件及redo log的打开、刷写模式，对于这个参数，文档上是这样描述的：
        有三个值：fdatasync(默认)，O_DSYNC，O_DIRECT
        参考： http://blog.csdn.net/dukope/article/details/9015539
        (2) 事务隔离模式        transaction-isolation = READ-UNCOMMITTED xxx
        (3) 读写线程数目可以增大
            innodb_read_io_threads = 8
            innodb_write_io_threads = 4

5. example

        测试服务器有 16GB的物理内存，假定其峰值最大的连接数为 20 个，表对象使用InnoDB 存储引擎，我们的内存参数如何配置呢？

        [client]
        port        = 3306 #客户端默认连接端口
        socket      = /tmp/mysql.sock  #用于本地连接的socket套接字

        [mysqld]  # 服务端基本配置
        port        = 3306 # mysql监听端口
        socket      = /tmp/mysql.sock #为MySQL客户端程序和服务器之间的本地通讯指定一个套接字文件

        具体配置如下：
        （1）、首先，为操作系统预留 20% 的内存，约为 3GB。
        （2）、与线程相关的几个关键参数设置如下：

        sort_buffer_size=2m
        read_buffer_size=2m
        read_rnd_buffer_size=2m
        join_buffer_size=2m

        max_prepared_stmt_count=500000
        max_connect_errors = 10
        table_open_cache = 2048
        max_allowed_packet = 16M
        binlog_cache_size = 16M
        max_heap_table_size = 64M
        thread_cache_size = 1000
        query_cache_size = 0
        query_cache_type = 0
        ft_min_word_len = 4
        thread_stack = 192K
        tmp_table_size = 64M

            （x）预计连接数达到峰值时，线程预计最大将有可能占用 20 *（2+2+2+2）= 160M内存（理论最大值）。 这里暂且不计！

        （3）、剩下的空间 16-3=13GB，就可以全部都分配给InnoDB 的缓存池，设定相关的参数如下：

        #innodb_data_file_path = ibdata1:100M:autoextend
        innodb_file_per_table = true
        innodb_buffer_pool_size=32G
        innodb_thread_concurrency=8
        innodb_flush_method=O_DIRECT
        innodb_log_buffer_size=16m
        innodb_flush_log_at_trx_commit=1

        innodb_file_per_table = true
        innodb_buffer_pool_instances=8
        innodb_log_file_size = 2G
        innodb_log_files_in_group = 2

        innodb_read_io_threads = 16
        innodb_write_io_threads = 16
        innodb_io_capacity = 20000
        innodb_io_capacity_max = 40000


6. 参考

        https://blog.csdn.net/crazyhacking/article/details/17137601

        https://segmentfault.com/a/1190000011687570

        https://yq.aliyun.com/articles/645565
------

#### mariadb 测试数据

    https://mariadb.org/performance-evaluation-of-mariadb-10-1-and-mysql-5-7-4-labs-tplc/


### mysql

1.编译

    cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/yealink/mysql \
        -DDEFAULT_CHARSET=utf8 \
        -DDEFAULT_COLLATION=utf8_general_ci \
        -DENABLED_LOCAL_INFILE=ON \
        -DWITH_INNODB_MEMCACHED=ON \
        -DWITH_SSL=system \
        -DWITH_INNOBASE_STORAGE_ENGINE=1 \
        -DWITH_FEDERATED_STORAGE_ENGINE=1 \
        -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
        -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
        -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
        -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
        -DCOMPILATION_COMMENT="zsd edition" \
        -DDOWNLOAD_BOOST=1 \
        -DMYSQL_UNIX_ADDR=/usr/local/yealink/mysqldata/3306/mysql.sock \
        -DSYSCONFDIR=/usr/local/yealink/mysqldata/3306 > mysql_cmake80.log 2>&1

2.参考

    https://www.cnblogs.com/zhangshengdong/p/9181650.html

3.MySQL基准测试和sysbench工具

    https://www.cnblogs.com/kismetv/p/7615738.html

<meta http-equiv="refresh" content="1">