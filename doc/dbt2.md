#### DBT2

- git

		https://sourceforge.net/projects/osdldbt/
		http://osdldbt.sourceforge.net/
	
	�ο�
		��˵��
		http://samurai-mysql.blogspot.com/2009/03/settingup-dbt-2.html
		https://blog.yannickjaquier.com/linux/dbt-2.html

		���ģ����޸�ָ��
		http://www.voidcn.com/article/p-smjxbggq-kz.html
		https://blog.csdn.net/bobpen/article/details/12775389

		��ϸ�Ŀɿ����̣�DBA Dojo
		https://dbadojo.com/2007/09/01/mysql-dbt2-benchmark-on-ec2-part-1/

		�쳣
		https://grokbase.com/t/postgresql/pgsql-performance/0926qbaxnm/cant-locate-test-parser-dbt2-pm-in-dbt2-tests
		https://sourceforge.net/p/osdldbt/mailman/message/6372253/
		https://sourceforge.net/p/osdldbt/mailman/message/28043065/
		http://www.voidcn.com/article/p-dglahawp-sr.html

		�ο�
		https://blog.csdn.net/jianwushuang/article/details/48140513

    ���Թ���

		1.��װ
		  ���������õ�perl�İ� Statistics::Descriptive, Test::Parser, Test::Reporter
		  centos 7��װperl��cpan���������ذ�װ����
		  yum install -y cpan
		  ������perl��-MCPAN -e shell��Ȼ�� cpan> install  xxx ��
          ���ߣ�
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
		  
		fix error: ��װ��expat�쳣�ģ��밲װ
		  yum install expat-devel -y
		�����쳣���ο�
		  http://www.voidcn.com/article/p-zeyopxdw-yp.html
		  https://stackoverflow.com/questions/13986282/xmlparser-refusing-to-install

		��װ�����õ�������
		yum -y install gnuplot gcc sysstat

		select TABLE_NAME,ENGINE,DATA_LENGTH,INDEX_LENGTH,DATA_LENGTH from information_schema.tables where TABLE_SCHEMA='dbt2';
		select table_name,sum(data_length) from tables where table_schema='dbt2' group by table_name with rollup;

		2.����
		[.]# cd dbt2-0.40
		[.]# mkdir my
		[.]#./configure --with-mysql --prefix=/root/tpcc/dbt2-0.40/my 
			--with-mysql-libs=/usr/local/mysql/lib --with-mysql-includes=/usr/local/mysql/include
		[.]#make -j12; make install

		3.����
		  ��������
			[.]#mkdir -p /tmp/dbt2-w5; ./src/datagen -w 5 -d /tmp/dbt2-w5 --mysql
			[.]#cd /root/tpcc/dbt2-0.40/scripts/mysql;
		  ����
			[.]#sh build_db.sh -d dbt2 -f /tmp/dbt2-w5 -u root -p mysql -h 10.200.110.163 
		  	�쳣�ο�
			�޸�build_db.sh LOCAL="LOCAL" 
			(https://stackoverflow.com/questions/3471474/mysql-load-data-infile-cant-get-stat-of-file-errcode-2)
			
		  ����sql���
			[.]# cd /root/tpcc/dbt2-0.40/storedproc/mysql
			[.]# mysql -u root -D dbt2 < new_order.sql;
			[.]# mysql -u root -D dbt2 < new_order_2.sql;
			[.]# mysql -u root -D dbt2 < order_status.sql;
			[.]# mysql -u root -D dbt2 < payment.sql;
			[.]# mysql -u root -D dbt2 < stock_level.sql;
		
		  ���в���
			[.]# cd /root/tpcc/dbt2-0.40//scripts
			[.]#./run_workload.sh -c 10 -t 10 -d 60 -w 5 -u root
		  �쳣��
			[.]#./post-process --dir /mnt/dbt2-0.40/scripts/output/20 --xml 		ʧ�ܰ���������
			�޷��ó������� ������driver.out ���ݻ��ǲ���
		