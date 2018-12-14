
#### 5. hammerdb test

- git

	https://www.hammerdb.com/document.html

#### 1. sysbench 1.0

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

#### 2. linkbench

		非TPCC!!!
		mariadb  use 
		给的数据结果
		https://mariadb.org/performance-evaluation-of-mariadb-10-1-and-mysql-5-7-4-labs-tplc/
		facebook 在线测试的


