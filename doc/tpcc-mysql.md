#### TPCC-Mysql
��Ҫ����mysql�Ĳ��ԣ�

- git

[tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)

- ����

	[��ϸָ��(����gnuplot��ͼ)](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
	
- �Բ����
	
		����
			CENTOS-7
		1 ��װ 
			mariadb(or mysql)
			��mariadb��·����ӵ�ϵͳPATH�� �� export PATH=(mariadb-path)/bin:$PATH
			���mariadb��·������ŵ�/etc/ld.so.conf����ִ�����ldconfig�� 
		2 ����
			git https://github.com/Percona-Lab/tpcc-mysql.git
		3 ����
			cd tpcc-mysql; cd src; make;
		4 ����
			mysqladmin  -uroot --password="" create tpcc10
		5 ����
			mysql -uroot --password="" tpcc10 < create_table.sql
		6 �����
			mysql -uroot --password="" tpcc10 < add_fkey_idx.sql
		7 ��������
			/*
			tpcc_load ����˵����
			Usage: tpcc_load -h server_host -P port -d database_name -u mysql_user -p mysql_password -w warehouses -l part -m min_wh -n max_wh
			* [part]: 1=ITEMS 2=WAREHOUSE 3=CUSTOMER 4=ORDERS*/

			./tpcc_load -h127.0.0.1 -dtpcc10 -uroot -p "" -w10
		8 ��ʼ����
			/*
			Usage: tpcc_start -h server_host -P port -d database_name -u mysql_user -p mysql_password -w warehouses -c connections -r warmup_time -l running_time -i report_interval -f report_file -t trx_file
			*/
			./tpcc_start -h127.0.0.1 -P3306 -dtpcc10 -uroot -w10 -c10 -r60 -l180 > tpcc10_w10_c10_r60_l180.log 2&>1
		9 �������
		 [0] sc:100589  lt:0  rt:0  fl:0
		 [1] sc:100552  lt:0  rt:0  fl:0
		 [2] sc:10059  lt:0  rt:0  fl:0
		 [3] sc:10057  lt:0  rt:0  fl:0
		 [4] sc:10058  lt:0  rt:0  fl:0
		 in 120 sec.
		 [0]New-Order���¶���ҵ��
		 [1]Payment��֧��ҵ��ͳ�ƣ�����ͬ��
		 [2]Order-Status������״̬ҵ��ͳ�ƣ�����ͬ��
		 [3]Delivery������ҵ��ͳ�ƣ�����ͬ��
		 [4]Stock-Level�����ҵ��ͳ�ƣ�����ͬ��
			
		 sc �ɹ�(success)������
		 lt �ӳ�(late)������
		 rt ����(retry)������
		 fl ʧ��(failure)����
		
		10 ��־��
		��������w = 10, c = 10, r = 60, l = 180
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
		���ݲ���Ȼ׼ȷ��������������

