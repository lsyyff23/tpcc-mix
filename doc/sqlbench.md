
#### sqlbench

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
