##mysql高级部分学习

###索引

####索引分类

- 单值索引：一个索引只包含单个列，一个表可以有多个单列索引
- 唯一索引：索引列必须唯一，不允许有空值
- 复合索引：一个索引包含多个列

#### 使用ALTER命令：

- ALTER TABLE mytable ADD PRIMARY KEY(colunm_list) 添加一个主键，要求必须唯一非空
- ALTER TABLE mytable ADD UNIQUE KEY(column_list) 唯一索引
- ALTER TABLE mytable ADD INDEX index_name(column_list) 普通索引
- ALTER TABLE mytable ADD FULLTEXT index_name(column_list)全文索引

#### 索引B+TREE树

![image-20191215153723880](/Users/huximing/Desktop/mysql高级部分学习.assets/image-20191215153723880.png)

### 索引优化：

##### where category_id=1 and count>1order by views desc

where---order ，内部变量建立联合索引，如果where中出现了范围查找仍然会抛弃后面order的联合索引。

建立三个变量的索引会出现索引失效的情况，此时只需要建立category_id和views的索引即可，跳过索引失效列。

##### 双表主外键，通过join关联索引添加位置

select * from class left join book on class.id=book.id;

ALTER table book add index Y(id);

由于左连接特性，left join的条件是确定如何从右边的表中搜索行，左边是一定都有的，因此**左连接加右边索引**，也就是book加索引，**右连接与左连接相反。**

三表与双表机制相同，均建立在非全表显示表中。

### 索引失效补充

- like一通配符%开头会使得索引失效（%JUM）右侧%不会失效。

**问题：解决like “%字符串%” 如何不失效。**

- 用覆盖索引，只给需要搜索的建立索引，查询的列也是索引列。

  例如:select name,age from user where name like '%aa%';建立name和age的复合索引，id本身就是主键索引替换select 后方的数据，**id，name，age，id name，id age， id name age索引都存在**。只要查询中有一个匹配上就可以，如上，如果只是name 或者单独的age再加其他的属性，例如主键索引，也可以用索引搜索。**但是**如果用* 或者添加了一个没有任何索引的列仍然会出现索引失效。问题

**索引失效第九条**，字符串不加单引号会索引失效。其原理和第三条一直，mysql数据库内部帮我们做了转化，例如2000，如果是字符串的2000 ，我们在输入的时候不加引号，mysql还是能帮助我们查找到数据，但是他是经过了内部的转化，将数字转化为字符串，因此会失效。

explain select * from test03 where c1='a1'and c2='a2' and c5='a5' order by c3,c2.**不会出现filesort**

c1~c4建立索引。此时索引不失效，虽然order不是按照顺序的，但是前方已经固定了c2，因此等同于只用了c3 一个，此时索引不失效。**如果上述语句中去除c2='a2'，此时sql索引失效。**

group by 分组之前必排序，会有临时表产生。

![image-20191217114848181](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217114848181.png)

### 索引优化

![image-20191217132032987](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217132032987.png)

```
show open tables；显示当前的表锁
```

![image-20191217154333212](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217154333212.png)

**总结MyISAM：读锁会阻塞写，但是不回阻塞读，而写锁会吧读写都阻塞，MyISAM的读写锁是写优先，因此不适合作为主表的写引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使得查询很难得到锁，造成阻塞。**

####事务：

![image-20191217155018270](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217155018270.png)

脏读：事务A读取到了事务B已修改的但尚未提交的数据。

幻读：事务A读取到事务B提交的新增数据。隔离性

不可重复读：一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变、或某些记录已经被删除了。隔离性

![image-20191217155655494](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217155655494.png)

默认是可重复读。

select ...for update 锁定当前选中的行，其他客户端无法操作该行



show status like 'innodb_row_lock%'

![image-20191217162816029](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217162816029.png)

状态说明：

![image-20191217162846416](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217162846416.png)

![image-20191217162949955](/Users/huximing/IdeaProjects/ziliao/workspace/mysql/mysql高级部分学习.assets/image-20191217162949955.png)

### mysql的inndb锁机制:Next-Key Lock解决幻读。

 数据库使用锁是为了支持更好的并发，提供数据的完整性和一致性。InnoDB是一个支持行锁的存储引擎，锁的类型有：共享锁（S）、排他锁（X）、意向共享（IS）、意向排他（IX）。为了提供更好的并发，InnoDB提供了非锁定读：不需要等待访问行上的锁释放，读取行的一个快照。该方法是通过InnoDB的一个特性：MVCC来实现的。

**InnoDB有三种行锁的算法：**

1，Record Lock：单个行记录上的锁。

2，Gap Lock：间隙锁，锁定一个范围，但不包括记录本身。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。

3，Next-Key Lock：1+2，锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。

```mysql
测试一：默认RR隔离级别

root@localhost : test 10:56:10>create table t(a int,key idx_a(a))engine =innodb;
Query OK, 0 rows affected (0.20 sec)

root@localhost : test 10:56:13>insert into t values(1),(3),(5),(8),(11);
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

root@localhost : test 10:56:15>select * from t;
+------+
| a    |
+------+
|    1 |
|    3 |
|    5 |
|    8 |
|   11 |
+------+
rows in set (0.00 sec)

section A:

root@localhost : test 10:56:27>start transaction;
Query OK, 0 rows affected (0.00 sec)

root@localhost : test 10:56:29>select * from t where a = 8 for update;
+------+
| a    |
+------+
|    8 |
+------+
row in set (0.00 sec)


section B:
root@localhost : test 10:54:50>begin;
Query OK, 0 rows affected (0.00 sec)

root@localhost : test 10:56:51>select * from t;
+------+
| a    |
+------+
|    1 |
|    3 |
|    5 |
|    8 |
|   11 |
+------+
rows in set (0.00 sec)

root@localhost : test 10:56:54>insert into t values(2);
Query OK, 1 row affected (0.00 sec)

root@localhost : test 10:57:01>insert into t values(4);
Query OK, 1 row affected (0.00 sec)

++++++++++
root@localhost : test 10:57:04>insert into t values(6);

root@localhost : test 10:57:11>insert into t values(7);

root@localhost : test 10:57:15>insert into t values(9);

root@localhost : test 10:57:33>insert into t values(10);
++++++++++
上面全被锁住，阻塞住了

root@localhost : test 10:57:39>insert into t values(12);
Query OK, 1 row affected (0.00 sec)
```

**问题：**

**为什么section B上面的插入语句会出现锁等待的情况**？InnoDB是行锁，在section A里面锁住了a=8的行，其他应该不受影响。why？

**分析：**

因为InnoDB对于行的查询都是采用了Next-Key Lock的算法，锁定的不是单个值，而是一个范围（GAP）。上面索引值有1，3，5，8，11，其记录的GAP的区间如下：是一个**左开右闭**的空间（原因是默认主键的有序自增的特性，结合后面的例子说明）

（-∞,1]，(1,3]，(3,5]，(5,8]，(8,11]，(11,+∞）

特别需要注意的是，InnoDB存储引擎还会对辅助索引下一个键值加上gap lock。如上面分析，那就可以解释了。

该SQL语句锁定的范围是（5,8]，下个下个键值范围是（8,11]，所以插入5~11之间的值的时候都会被锁定，要求等待。即：插入5，6，7，8，9，10 会被锁住。插入非这个范围内的值都正常。

因为例子里没有主键，所以要用隐藏的ROWID来代替，数据根据Rowid进行排序。而Rowid是有一定顺序的（自增），所以其中11可以被写入，5不能被写入，

###innodb和myisam的区别：

**MyISAM:**

myisam只支持表级锁，用户在操作myisam表时，select，update，delete，insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。也可以通过lock table命令来锁表，这样操作主要是可以模仿事务，但是消耗非常大，一般只在实验演示中使用。

**InnoDB ：**

Innodb支持事务和行级锁，是innodb的最大特色。

事务的ACID属性：atomicity,consistent,isolation,durable。

并发事务带来的几个问题：更新丢失，脏读，不可重复读，幻读。

事务隔离级别：未提交读(Read uncommitted)，已提交读(Read committed)，可重复读(Repeatable read)，可序列化(Serializable)