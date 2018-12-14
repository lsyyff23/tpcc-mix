-----------------------------
------------------------------
#                                TPC-C
------------------------------------
-------------------------------------

### TPC

[TPC](http://www.tpc.org/default.asp)是一个非盈利性组织机构，致力于事务性处理和数据基准测试、及向行业发布客观的、可验证的TPC性能数据。
  
### TPC-C

- 简介

TPC-C 是TPC关于OLTP测试的改进版本，于1992-06月发布，目前版本为5.11.0.

作为一个OLTP系统压测模型，TPC-C模拟了一个完整的环境，有大量终端在同个数据库操作不同事务。

- 模型

在TPC-C 业务模型里，模拟了一个公司销售系统，有N个仓库，每个仓库为10个区域供货，每个区域负责3000客户，每个区域提供的操作包括：建新订单、支付订单、发货处理、查询订单状态、查看库存；其中建议比例为 45：43：4：4：4。

- 评测指标

	tpm-C： transaction per minute, C为TPC的C套标准

	Price/tpmC: 性价比 Price/Performance，整套系统价格（软硬件+维护等综合价格）与tpm-C的比

### 测试工具  

TPCC的测试工具大都比较早，主要有Tpcc-mysql、DBT2、HammerDB、benchmarkSQL、sqlbench等，以下对这些一一说明：
（参考:[汇总](https://blog.csdn.net/luke_wang/article/details/71860667))	
 
#### 1. TPCC-Mysql
主要用于mysql的测试；

- git
	 
	`https://github.com/Percona-Lab/tpcc-mysql`

- 流程

	[详细指导(包括gnuplot画图)](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
	
- 自测过程
	
		环境
			CENTOS-7
		1 安装 
			mariadb(or mysql)
			将mariadb的路径添加到系统PATH， 如 export PATH=(mariadb-path)/bin:$PATH
			添加mariadb库路径，如放到/etc/ld.so.conf，再执行命令：ldconfig。 
		2 下载
			git https://github.com/Percona-Lab/tpcc-mysql.git
		3 编译
			cd tpcc-mysql; cd src; make;
		4 建库
			mysqladmin  -uroot --password="" create tpcc10
		5 建表
			mysql -uroot --password="" tpcc10 < create_table.sql
		6 建外键
			mysql -uroot --password="" tpcc10 < add_fkey_idx.sql
		7 载入数据
			/*
			tpcc_load 参数说明：
			Usage: tpcc_load -h server_host -P port -d database_name -u mysql_user -p mysql_password -w warehouses -l part -m min_wh -n max_wh
			* [part]: 1=ITEMS 2=WAREHOUSE 3=CUSTOMER 4=ORDERS*/

			./tpcc_load -h127.0.0.1 -dtpcc10 -uroot -p "" -w10
		8 开始测试
			/*
			Usage: tpcc_start -h server_host -P port -d database_name -u mysql_user -p mysql_password -w warehouses -c connections -r warmup_time -l running_time -i report_interval -f report_file -t trx_file
			*/
			./tpcc_start -h127.0.0.1 -P3306 -dtpcc10 -uroot -w10 -c10 -r60 -l180 > tpcc10_w10_c10_r60_l180.log 2&>1
		9 结果分析
		 [0] sc:100589  lt:0  rt:0  fl:0
		 [1] sc:100552  lt:0  rt:0  fl:0
		 [2] sc:10059  lt:0  rt:0  fl:0
		 [3] sc:10057  lt:0  rt:0  fl:0
		 [4] sc:10058  lt:0  rt:0  fl:0
		 in 120 sec.
		 [0]New-Order，新订单业务
		 [1]Payment，支付业务统计，其他同上
		 [2]Order-Status，订单状态业务统计，其他同上
		 [3]Delivery，发货业务统计，其他同上
		 [4]Stock-Level，库存业务统计，其他同上
			
		 sc 成功(success)次数，
		 lt 延迟(late)次数，
		 rt 重试(retry)次数，
		 fl 失败(failure)次数
		
		10 日志：
		用例条件w = 10, c = 10, r = 60, l = 180
		mariadb server:
			cpu: 24 Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz
			Mem: 15884
			disk: 248G

		STOPPING THREADS..........

		<Raw Results>
		  [0] sc:4 lt:2254  rt:0  fl:0 avg_rt: 452.4 (5)
		  [1] sc:323 lt:1933  rt:0  fl:0 avg_rt: 267.0 (5)
		  [2] sc:134 lt:91  rt:0  fl:0 avg_rt: 207.0 (5)
		  [3] sc:3 lt:222  rt:0  fl:0 avg_rt: 2587.6 (80)
		  [4] sc:46 lt:180  rt:0  fl:0 avg_rt: 608.1 (20)
		 in 180 sec.
		
		<Raw Results2(sum ver.)>
		  [0] sc:4  lt:2255  rt:0  fl:0
		  [1] sc:323  lt:1933  rt:0  fl:0
		  [2] sc:134  lt:91  rt:0  fl:0
		  [3] sc:3  lt:222  rt:0  fl:0
		  [4] sc:46  lt:180  rt:0  fl:0
		
		<Constraint Check> (all must be [OK])
		 [transaction percentage]
		        Payment: 43.47% (>=43.0%) [OK]
		   Order-Status: 4.34% (>= 4.0%) [OK]
		       Delivery: 4.34% (>= 4.0%) [OK]
		    Stock-Level: 4.35% (>= 4.0%) [OK]
		 [response time (at least 90% passed)]
		      New-Order: 0.18%  [NG] *
		        Payment: 14.32%  [NG] *
		   Order-Status: 59.56%  [NG] *
		       Delivery: 1.33%  [NG] *
		    Stock-Level: 20.35%  [NG] *
		
		<TpmC>
		                 752.667 TpmC
		数据不尽然准确，待分析！！！


#### 2. sqlbench

淘宝内部，C语言实现的TPC-C测试工具，声明支持postgresql+mysql，这里测试mysql时编译失败！！


		
		
- GIT源码

		https://github.com/swida/sqlbench
	

- 测试过程
		
	1. 环境搭建
	
		  	安装postgres 
				略
          	编译sqlbench
				cd ..../sqlbench/
				mkdir pg -p
				./configure --with-postgresql --prefix=/root/tpcc/sqlbench/pg
				make j5; make install

	2. 测试
	
			0 vi env.sh
				将后面命令需要的环境添加到这里，
				这里不采用这种方式，都是命令直接使用

			1 create database
				PGUSER=postgres PGPASSWORD=postgres PGHOST=dbca PGPORT=5432 DBT2DBNAME=testdb  bin/create-tables
	
			2 load database
				PGPASSWORD=postgres bin/datagen -t postgresql --dbname=testdb --host=10.200.110.163 --port=5432 --user=postgres -j10 -w1
			
			3 create index
				PGUSER=postgres PGPASSWORD=postgres PGHOST=10.200.110.163 PGPORT=5432 DBT2DBNAME=testdb bin/create-indexes
	
			4 analyze
				psql postgres://postgres:postgres@10.200.110.163:5432/testdb <<EOF
				VACUUM analyze customer;  
				VACUUM analyze district;  
				vacuum analyze history;  
				vacuum analyze item;  
				vacuum analyze new_order;  
				vacuum analyze order_line;  
				vacuum analyze orders;  
				vacuum analyze stock;  
				vacuum analyze warehouse;  
				checkpoint;  
				EOF
		
			5 plan out
			  PGUSER=postgres PGPASSWORD=postgres PGHOST=10.200.110.163 PGPORT=5432 DBT2DBNAME=testdb bin/plans -o plans.out
	
			6 benchmark
				bin/sqlbench -t postgresql --dbname=testdb --user=postgres --host=10.200.110.163 --port=5432 --no-thinktime -w1 --sleep 10 -c10 -l60 -r5 --sqlapi extended
	
			7 process log 
				bin/post_process -l mix.log

			8 结果为：
				                         Response Time (s)
				 Transaction      %    Average :    90th %        Total        Rollbacks      %
				------------  -----  ---------------------  -----------  ---------------  -----
				    Delivery   3.63      0.032 :     0.048          171                0   0.00
				   New Order  45.09      0.019 :     0.039         2127               23   1.08
				Order Status   4.09      0.001 :     0.001          193                0   0.00
				     Payment  43.04      0.007 :     0.017         2030                0   0.00
				 Stock Level   4.16      0.002 :     0.002          196                0   0.00
				------------  -----  ---------------------  -----------  ---------------  -----
				
				2127.00 new-order transactions per minute (NOTPM)
				1.0 minute duration
				0 total unknown errors
				5 second(s) ramping up

	NOTE
 
	    postgres test ok
		mariadb , build failed undefine reference to 'cli_safe_read' ... 
			  from libmariadbclient-dev.so ,  build the lib failed !!!!
- 参考

	http://mysql.taobao.org/monthly/2018/07/09/

	https://github.com/digoal/blog/blob/master/201805/20180530_01.md   

 
#### 3. DBT2

- git

		https://sourceforge.net/projects/osdldbt/
		http://osdldbt.sourceforge.net/
	
	参考
		简单说明
		http://samurai-mysql.blogspot.com/2009/03/settingup-dbt-2.html
		https://blog.yannickjaquier.com/linux/dbt-2.html

		中文，有修改指导
		http://www.voidcn.com/article/p-smjxbggq-kz.html
		https://blog.csdn.net/bobpen/article/details/12775389

		详细的可靠流程，DBA Dojo
		https://dbadojo.com/2007/09/01/mysql-dbt2-benchmark-on-ec2-part-1/

		异常
		https://grokbase.com/t/postgresql/pgsql-performance/0926qbaxnm/cant-locate-test-parser-dbt2-pm-in-dbt2-tests
		https://sourceforge.net/p/osdldbt/mailman/message/6372253/
		https://sourceforge.net/p/osdldbt/mailman/message/28043065/
		http://www.voidcn.com/article/p-dglahawp-sr.html

		参考
		https://blog.csdn.net/jianwushuang/article/details/48140513

    测试过程

		1.安装
		  分析数据用到perl的包 Statistics::Descriptive, Test::Parser, Test::Reporter
		  centos 7安装perl的cpan，用于下载安装包；
		  yum install -y cpan
		  用命令perl　-MCPAN -e shell，然后 cpan> install  xxx ，
          或者：
		  cpan Statistics::Descriptive
		  cpan GnuPG::Interface
		  cpan Chart::Graph::Gnuplot
		  cpan XML::Simple 
		  cpan XML::Twig
		  cpan XML::SAX
		  cpan XML::SAX::Expat
		  cpan XML::Simple
		  cpan Test::Parser
		  cpan Test::Reporter
		  
		fix error: 安装包expat异常的，请安装
		  yum install expat-devel -y
		其他异常，参考
		  http://www.voidcn.com/article/p-zeyopxdw-yp.html
		  https://stackoverflow.com/questions/13986282/xmlparser-refusing-to-install

		安装运行用到的命令
		yum -y install gnuplot gcc sysstat

		select TABLE_NAME,ENGINE,DATA_LENGTH,INDEX_LENGTH,DATA_LENGTH from information_schema.tables where TABLE_SCHEMA='dbt2';
		select table_name,sum(data_length) from tables where table_schema='dbt2' group by table_name with rollup;

		2.编译
		[.]# cd dbt2-0.40
		[.]# mkdir my
		[.]#./configure --with-mysql --prefix=/root/tpcc/dbt2-0.40/my 
			--with-mysql-libs=/usr/local/mysql/lib --with-mysql-includes=/usr/local/mysql/include
		[.]#make -j12; make install

		3.测试
		  生成数据
			[.]#mkdir -p /tmp/dbt2-w5; ./src/datagen -w 5 -d /tmp/dbt2-w5 --mysql
			[.]#cd /root/tpcc/dbt2-0.40/scripts/mysql;
		  建表
			[.]#sh build_db.sh -d dbt2 -f /tmp/dbt2-w5 -u root -p mysql -h 10.200.110.163 
		  	异常参考
			修改build_db.sh LOCAL="LOCAL" 
			(https://stackoverflow.com/questions/3471474/mysql-load-data-infile-cant-get-stat-of-file-errcode-2)
			
		  导入sql语句
			[.]# cd /root/tpcc/dbt2-0.40/storedproc/mysql
			[.]# mysql -u root -D dbt2 < new_order.sql;
			[.]# mysql -u root -D dbt2 < new_order_2.sql;
			[.]# mysql -u root -D dbt2 < order_status.sql;
			[.]# mysql -u root -D dbt2 < payment.sql;
			[.]# mysql -u root -D dbt2 < stock_level.sql;
		
		  运行测试
			[.]# cd /root/tpcc/dbt2-0.40//scripts
			[.]#./run_workload.sh -c 10 -t 10 -d 60 -w 5 -u root
		  异常点
			[.]#./post-process --dir /mnt/dbt2-0.40/scripts/output/20 --xml 		失败啊！！！！
			无法得出分析， 但看了driver.out 数据还是不好
		
#### 4. benchmarkSQL

- git

		编写语言:java
		官网： https://bitbucket.org/openscg/benchmarksql
 
		github：	https://github.com/angoca/BenchmarkSQL

		支持： oracle pg firebird not mysql

	 	NOTE: 5.0 支持oracle firebird postgres , 无mysql!!!!

- Download

		官方：https://sourceforge.net/projects/benchmarksql/
	
		这里为了支持mysql，下载的是：https://github.com/waterguo/benchmarksql
			Currently the lastest version is 4.1.1
			There is also a fork of 3.0.9 with MySQL support in the history

- 测试过程
		

		安装ant
			https://www.jianshu.com/p/e5157e7eb553
			
		下载apache ant
			https://ant.apache.org/bindownload.cgi
		添加环境
			vi /etc/profile
			export ANT_HOME=/usr/local/ant
			export PATH=$PATH:$ANT_HOME/bin

		下载源码
			https://github.com/waterguo/benchmarksql

		编译代码
			cd ..../benchmarksql
			ant
		
		测试参考
			HOW-TO-RUN.txt和淘宝

	    postgres：
			1 modify props.pg
			warehouses=1
		
			2 create table/schema
			./runSQL.sh props.pg sqlTableCreates
			postgres >\dn
	
			3 load data
				a> direct into database
				./runLoader.sh props.pg numWarehouses 1 
				
				b> save to csv then copy to database ,  fileLocation is the same in sqlTableCopies
				./runLoader.sh props.pg numWarehouses 1 fileLocation /root/tpcc/benchmarksql/run/dt/
	            ./runSQL.sh props.pg sqlTableCopies 

			4 create index
			./runSQL.sh props.pg sqlIndexCreates
	
			5 runBenchmark
			./runBenchmark.sh props.pg	

	 	 mysql:			
			1 modify props.mysql
			warehouses=1
		
			2 create table/schema
			mysql> create database benchmarksql;
			./runSQL.sh props.mysql sqlTableCreates.mysql
	
			3 generate CSV
			./runLoader.sh props.mysql numWarehouses 1 fileLocation /root/tpcc/benchmarksql-mysql/run/dt/
	
				(not NEED 4 copy to mysql
						 	./runSQL.sh props.mysql sqlTableCopies.mysql)

			4 create index
			./runSQL.sh props.mysql sqlIndexCreates.mysql
	
			5 runBenchmark
			./runBenchmark.sh props.mysql	

		pgxl:
			[fix distribute by]https://sourceforge.net/p/postgres-xl/tickets/67/

- 修改了2个bug

		已提交至github: https://github.com/waterguo/benchmarksql

- reference
		
		1.https://github.com/rmpvilaca/BenchmarkSQLHBase
		2.淘宝 BenchmarkSQL postgresql 9.5 使用介绍，优化pg! 参考该文章
		  http://mysql.taobao.org/monthly/2016/02/07/
		3.some modify 
		  https://github.com/andl/benchmarkSQL


#### 5. hammerdb test

- git

	https://www.hammerdb.com/document.html

#### 6. sysbench 1.0

Lua编写的测试工具

- git

		https://github.com/digoal/blog/blob/master/201809/20180913_01.md

- 介绍

		https://www.percona.com/blog/2018/03/05/tpcc-like-workload-sysbench-1-0/
		TPCC-Like Workload for Sysbench 1.0
		https://github.com/Percona-Lab/sysbench
		https://github.com/akopytov/sysbench

- NOTE
	
		OLTP测试 只提供
		read:                            938224    -- 读总数
        write:                           268064    -- 写总数
        other:                           134032    -- 其他操作总数(SELECT、INSERT、UPDATE、DELETE之外的操作，例如COMMIT等)
        total:                           1340320    -- 全部总数
		并非标准的TPCC测试

#### 7. linkbench

		非TPCC!!!
		mariadb  use 
		给的数据结果
		https://mariadb.org/performance-evaluation-of-mariadb-10-1-and-mysql-5-7-4-labs-tplc/
		facebook 在线测试的




参考
	
	1.https://blog.csdn.net/notbaron/article/details/78080049
    2.[<<TPC-C 基准测试程序在 OLTP 系统中的应用与实现>>](https://www.ibm.com/developerworks/cn/data/library/db-cn-tpc-ctest/index.html)
	3.http://blog.sina.com.cn/s/blog_4485748101019wsh.html
