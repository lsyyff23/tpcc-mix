#### DBT2

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
		