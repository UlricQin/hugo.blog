+++
date = "2011-10-03T19:52:20+08:00"
menu = "main"
tags = ["mysql", "interview"]
title = "对比MySQL的存储引擎：InnoDB vs MyISAM"

+++

互联网公司用得最多的关系型数据库估计就是MySQL了，研发岗面试的时候被问到MySQL的问题无可厚非。

MySQL有两个很有名、很常用的存储引擎：InnoDB和MyISAM，请做一下比较

整体来看，MyISAM的查询性能更好一些，InnoDB支持数据库事务、外键等高级特性，更新操作更快一些

具体我们从以下方面说明：

- 1、事务支持：InnoDB支持事务，MyISAM不支持
- 2、参照完整性：即外键，InnoDB支持，MyISAM不支持
- 3、FULLTEXT类型索引：MyISAM支持，InnoDB不支持，当然了，其实用数据库做全文索引还是比较少见，通常的选择是Lucene之类的
- 4、表行数计算：InnoDB 中不保存表的具体行数，也就是说，执行`select count(*) from table`时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当`count(*)`语句包含 where条件时，两种表的操作是一样的。
- 5、锁：MyISAM支持表锁，InnoDB既支持表锁也支持行锁，一个update语句，显然行锁比表锁效率高，但如果在执 行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表，例如`update table set num=1 where name like “%aaa%”`


最主要的估计就是上面几条了，还有一些从开发角度来看没那么重要的区别，比如：

- 1、MyISAM表在磁盘上表现为三个文件：.frm（存放表结构）.MYD（存放表数据）.MYI（存放表索引），备份恢复相对比较方便；而InnoDB就相对麻烦一些啦
- 2、LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性（例如外键）的表不适用
- 3、对AUTO_INCREMENT处理有差异，这个对开发和运维都没有太大差别，不详细说了，众位可以Google一下

嗯，个人感觉，能答出这些，面试官应该可以满意了吧，祝好运：）
