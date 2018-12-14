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

TPCC的测试工具大都比较早，主要有[Tpcc-mysql](./tpcc-mysql.md)、[DBT2](./dbt2.md)、[HammerDB](./others.md)、[benchmarkSQL](./benmarksql.md)、[sqlbench](./sqlbench.md)等，以下对这些一一说明：
（参考:[汇总](https://blog.csdn.net/luke_wang/article/details/71860667))	



参考
	
	1.https://blog.csdn.net/notbaron/article/details/78080049
    2.[<<TPC-C 基准测试程序在 OLTP 系统中的应用与实现>>](https://www.ibm.com/developerworks/cn/data/library/db-cn-tpc-ctest/index.html)
	3.http://blog.sina.com.cn/s/blog_4485748101019wsh.html
