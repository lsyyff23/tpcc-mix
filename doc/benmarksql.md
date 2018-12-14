#### 4. benchmarkSQL

- git

		��д����:java
		������ https://bitbucket.org/openscg/benchmarksql
 
		github��	https://github.com/angoca/BenchmarkSQL

		֧�֣� oracle pg firebird not mysql

	 	NOTE: 5.0 ֧��oracle firebird postgres , ��mysql!!!!

- Download

		�ٷ���https://sourceforge.net/projects/benchmarksql/
	
		����Ϊ��֧��mysql�����ص��ǣ�https://github.com/waterguo/benchmarksql
			Currently the lastest version is 4.1.1
			There is also a fork of 3.0.9 with MySQL support in the history

- ���Թ���
		

		��װant
			https://www.jianshu.com/p/e5157e7eb553
			
		����apache ant
			https://ant.apache.org/bindownload.cgi
		��ӻ���
			vi /etc/profile
			export ANT_HOME=/usr/local/ant
			export PATH=$PATH:$ANT_HOME/bin

		����Դ��
			https://github.com/waterguo/benchmarksql

		�������
			cd ..../benchmarksql
			ant
		
		���Բο�
			HOW-TO-RUN.txt���Ա�

	    postgres��
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

- �޸���2��bug

		���ύ��github: https://github.com/waterguo/benchmarksql

- reference
		
		1.https://github.com/rmpvilaca/BenchmarkSQLHBase
		2.�Ա� BenchmarkSQL postgresql 9.5 ʹ�ý��ܣ��Ż�pg! �ο�������
		  http://mysql.taobao.org/monthly/2016/02/07/
		3.some modify 
		  https://github.com/andl/benchmarkSQL

