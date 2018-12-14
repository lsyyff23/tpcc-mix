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

