### Mysql 学习记录

#### 1.join 的不同?

![image-20210228200348470](https://i.loli.net/2021/04/05/UtxCwrlZFdNGEAR.png)

#### 2.SQL语句分类

https://blog.csdn.net/jerrytomcat/article/details/82250898

	* DDL(Data Definition Language)数据定义语言.example: create,drop,alter,对数据的逻辑结构有修改
	* DQL(Data Query Language) 数据查询语言. example: select   查询操作
	* DML(Data Manipulation Language) 数据操作语言. example: insert,update,delete 对数据进行操作,增删改
	* DCL(Data Control Language) 数据控制语言.example:grank,revoke 对数据安全的操作,权限控制
	* TCL (Transaction Control Language) 事务控制语言 example: savepoint,rollback,set transaction

![img](https://i.loli.net/2021/04/05/YkwtNz3aspfdAe5.png)

#### 3.事务四大特性

ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）

- 原子性：不可分割的操作单元，事务中所有操作，要么全部成功；要么撤回到执行事务之前的状态
- 一致性：如果在执行事务之前数据库是一致的，那么在执行事务之后数据库也还是一致的；
- 隔离性：事务操作之间彼此独立和透明互不影响。事务独立运行。这通常使用锁来实现。一个事务处理后的结果，影响了其他事务，那么其他事务会撤回。事务的100%隔离，需要牺牲速度。
- 持久性：事务一旦提交，其结果就是永久的。即便发生系统故障，也能恢复。

#### 4.不同隔离级别会产生的问题

* 脏读（Dirty Read）事务执行过程中，B事务读取了A事务的修改。但是由于某些原因，A事务可能没有完成提交，发生RollBack了操作，则B事务所读取的数据就会是不正确的。
* 不可重复读（Nonrepeatable Read）B事务读取了两次数据，在这两次的读取过程中A事务修改了数据，B事务的这两次读取出来的数据不一样。
* 幻读（Phantom Read）B事务读取了两次数据，在这两次的读取过程中A事务添加了数据，B事务的这两次读取出来的集合不一样。innodb基于mvcc基本没有幻读的问题
* 第一类丢失更新,由于某个事务的回滚操作，参与回滚的旧数据将其他事务的数据更新覆盖了。
* 第二类丢失更新,多个事务同时更新一行数据导致的问题.
  **不可重复读重点在于update和delete，而幻读的重点在于insert，幻读，**不能通过行锁来避免****

#### ![20160319184334938](https://i.loli.net/2021/04/05/YH7DZlyhNbjX3VL.png)

#### 5.事务隔离级别

查看日志隔离级别:show variables like 'transaction_isolation';

MySQL数据库(InnoDB引擎)默认使用可重复读（ Repeatable read)

事物隔离分为4个级别

- read uncommitted(读未提交)是指，一个事务还没提交时，它做的变更就能被别的事务看到。

- read committed,(读提交)是指，一个事务提交之后，它做的变更才会被其他事务看到。

- repeatable read,(可重复读)是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。

- serializable(串行化)，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

  下面为举例:

  ![事务](https://i.loli.net/2021/04/05/m8CZRhs4NOeixFt.png)

- 若隔离级别是“读未提交”， 则 V1 的值就是 2。这时候事务 B 虽然还没有提交，但是结果已经被 A 看到了。因此，V2、V3 也都是 2。
- 若隔离级别是“读提交”，则 V1 是 1，V2 的值是 2。事务 B 的更新在提交后才能被 A 看到。所以， V3 的值也是 2。
- 若隔离级别是“可重复读”，则 V1、V2 是 1，V3 是 2。之所以 V2 还是 1，遵循的就是这个要求：事务在执行期间看到的数据前后必须是一致的。
- 若隔离级别是“串行化”，则在事务 B 执行“将 1 改成 2”的时候，会被锁住。直到事务 A 提交后，事务 B 才可以继续执行。所以从 A 的角度看， V1、V2 值是 1，V3 的值是 2。

#### 6.锁  悲观锁和乐观锁

```
悲观锁认为被它保护的数据是极其不安全的，每时每刻都有可能变动，一个事务拿到悲观锁后（可以理解为一个用户），其他任何事务都不能对该数据进行修改，只能等待锁被释放才可以执行。
加悲观锁的方法就是，在后面加上for update(oracle 是加for update no wait),　　select for update
```

```
乐观锁的“乐观情绪”体现在，它认为数据的变动不会太频繁。因此，它允许多个事务同时对数据进行变动。
通过版本进行控制,给表结构加一个version的字段，当修改一条数据时：
1.首先会进行比对当前获取到的version和数据库中的版本是否相同，不同则修改失败
2.如果版本相同,则会进行数据修改和version=version+1
```

#### 7.LBCC(Lock-Base Concurrent Control)

##### 共享锁(share locks)又名读锁 

​		是行锁的一种,对某一资源加共享锁,自身可以读该资源，其他人也可以读．但是修改需要等待所有的共享锁都释放

​		加锁:
​		select * from table_name lock in share mode

​		释放锁：

​		commit,rollback

排它锁(exclusive locks) 又名写锁

​		是行锁的一种,	对某一资源加排它锁，自身可以进行增删改差,其它事务无法进行任何操作,排它锁不能与其他锁并存．

​		加锁:

​			自动: DML语句会默认加排它锁

​			手动: select * from user where id=1 for update;

​			释放锁：

​			commit,rollback

意向锁

​	是一种表锁

间隙锁(gap locks)

​		根据数据库中的已有数据根据左开右闭的原则添加间隙锁

​		加锁:
​		select * from id>6 and id<15 for update

​		释放锁：

​		commit,rollback

#### 8.数据库三范式

##### 1.第一范式(确保每列保持原子性)（1NF）

数据库表中的所有字段值都是不可分解的原子值

##### 2.第二范式(确保表中的每列都和主键相关)

一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中

##### 3.第三范式(确保每列都和主键列直接相关,而不是间接相关)

每一列数据都和主键直接相关，而不能间接相关

#### 9.SQL语句多个字段执行顺序

一个查询语句同时出现了where,group by,having,order by的时候，执行顺序和编写顺序是：

1.执行where xx对全表数据做筛选，返回第1个结果集。

2.针对第1个结果集使用group by分组，返回第2个结果集。

3.针对第2个结果集中的每1组数据执行select xx，有几组就执行几次，返回第3个结果集。

4.针对第3个结集执行having xx进行筛选，返回第4个结果集。

5.针对第4个结果集排序。

**通过一个顺口溜总结下顺序：我(W)哥(G)是(SH)偶(O)像**。按照执行顺序的关键词首字母分别是W（where）->G（Group）->S（Select）->H（Having）->O（Order），对应汉语首字母可以**编成容易记忆的顺口溜：我(W)哥(G)是(SH)偶(O)像**

#### 10.B+树索引和哈希索引

哈希索引支持=查询，不支持范围查询如>或<

#### 2、Mysql中的myisam与innodb的区别

1、InooDB支持事务，而MyISAM不支持事务

2、InnoDB支持行级锁，而MyISAM支持表级锁

3、InnoDB支持MVCC，而MyISAM不支持

4、InnoDB支持外键，而MyISAM不支持

5、InnoDB不支持全文索引，而MyISAM支持

#### 11.MVCC(多版本并发控制) 

MVCC 只适用于读已提交,可重复读两个隔离级别，

每一事务都会分配一个唯一的事务id

隐藏字段

DB_TRX_ID 事务id

DB_ROLL_PTR 指向undolog中的一个指针

DB_ROW_ID 

![image-20210301143530953](https://i.loli.net/2021/04/05/kyBOcteHojKxd18.png)

![image-20210301143641686](https://i.loli.net/2021/04/05/PBu3jhSg6KcGdiZ.png)

#### 12.如果不声明主键会有那些危害

1.行锁变表锁

2.自增超过2^32 -1会归0，覆盖

3.会创建一个_rowid int 6byte 造成资源浪费．



