#### TPCC-Mysql
主要用于mysql的测试；

- git

[tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)

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

