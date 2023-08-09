## MySQL锁

### 1. 锁简介

数据库中，每时每刻都在发生锁。

#### 1.1 锁分类

通常分为：

- 共享锁（shared lock）： 也叫读锁（read lock），互相不阻塞，互不干扰。
- 排他锁（exclusive lock）：也叫写锁（write lock），会阻塞其他的写锁和读锁。

锁粒度分：

- 全局锁：锁整个Database，由MySQL的SQL layer层实现
- 表级锁：锁Table，由MySQL的SQL layer层实现
- 行级锁：锁Row，由存储引擎实现。

### 2. 全局锁

全局锁是对整个数据库实例加锁，加锁后整个实例处于只读状态，后续的DML、DDL语句、已经更新操作的事务提交都被阻塞。

典型场景，对全库做逻辑备份。

加全局锁命令

```mysql
flush tables with read lock;
```

释放全局锁命令

```mysql
unlock tables;
```

或者断开加锁session的连接，自动释放全局锁。

对于InnoDB，使用mysqldump备份时可以使用--single-transaction参数，利用mvcc提供一致性视图，而不使用全局锁，不会影响业务的正常运行。

### 3. 表级锁

#### 3.1 表读写锁

表读锁，Table Read Lock。表写锁，Table Write Lock。

查看表锁状态

```mysql
mysql> show status like 'table%';
```

- table_locks_immediate：产生表级锁定的次数
- table_locks_waited：出现表级锁竞争等待的次数

手动加表锁

```mysql
lock table t1 read:
lock table t2 write;
```

手动释放表锁

```mysql
unlock tables;
```

#### 3.2 元数据锁

元数据锁（meta data lock，MDL），不需要显式指定，在访问一个表的时候自动加上，保证读写的正确性。

对表做增删改查，加元数据读锁；对表结构变更，加元数据写锁。

#### 3.3 自增锁

自增锁（AUTO-INC Lock），发生涉及`auto_increment`列的事务性插入操作时产生。

### 4. 行级锁

主要是InnoDB的行级锁，通过给索引上的索引项加锁来实现。只有通过索引条件检索的数据，InnoDB才使用行级锁，否则，使用表锁。

对于增删改语句，会自动加写锁，普通的查询语句，不会加任何锁。事务可以通过命令手动加锁

```mysql
#读锁
select * from t1 where id =1 lock in share mode;
#写锁
select * from t1 where id =1 for update;
```

#### 4.1 记录锁

记录锁（Record Lock）锁住索引记录的一行，在单条索引记录加锁。

记录锁永远锁的是索引，而非记录本身，即使表中没有显式索引，也会创建一个隐藏的聚簇索引，那么锁住的就是隐藏的聚簇索引。

#### 4.2 间隙锁

间隙锁（Gap Lock）锁住一个索引区间，开区间，不包含两端端点。

间隙锁可防止幻读，保证索引间隙不会插入数据。

在可重复读（Repeatable Read）下生效。

例如：一个表数据，id是主键索引，var是普通索引。

| id   | var  |
| ---- | ---- |
| 1    | 10   |
| 2    | 20   |
| 4    | 30   |
| 8    | 40   |

session1

```mysql
begin;
select * from t1 where id > 4 for update; -- 间隙锁
commit;
```

session2

```mysql
begin;
insert into t1 values(6,50); -- 阻塞
insert into t1 values(3,50); -- 成功
commit;
```

#### 4.3 临键锁

临键锁（Next-Key Lock）相当于记录锁+间隙锁，左开右闭区间。

默认情况下，InnoDB使用临键锁，不同场景下会退化：

- 唯一索引等值，且记录存在，记录锁
- 唯一索引等值，且记录不存在，间隙锁
- 唯一索引范围，临键锁
- 非唯一索引，临键锁

seesion1

```mysql
begin;
select * from t1 where var =30 for update;
-- var非唯一索引，临键锁(20,30],(30,40]
commit;
```

session2

```mysql
begin;
insert into t1 values (6,25); -- 阻塞
select * from t1 where var = 20 for update; -- 阻塞
insert into t1 values (6,35); -- 阻塞
insert into t1 values (6,41); -- 成功
```

#### 4.4 插入意向锁

插入意向锁（Insert Intention Lock）是一种在insert操作之前设置的特殊的间隙锁。

插入意向锁表示一种插入意图，当多个不同的事务，同时往同一个索引的同一个间隙插入数据，互相不会阻塞。

插入意向锁会互斥其他间隙写锁、记录锁。

```mysql
# session1
begin;
insert into t1 values (5,35); -- 成功
# session2
begin;
insert into t1 values (6,36); -- 成功
```

两个语句都成功，说明插入意向锁之间不互斥。

```mysql
# session1
begin;
select * from t1 where id > 4 for update;
# session2
begin;
insert into t1 values (7,37); -- 阻塞
```

插入意向锁阻塞，说明和其他写锁互斥。

通过`show engine innodb status\G  `命令查看InnoDB引擎状态。可以看到最后，加了插入意向锁。

```mysql
insert into t1 values(7,37)
------- TRX HAS BEEN WAITING 25 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 29 page no 4 n bits 80 index PRIMARY of table `test`.`t1` trx id 2370 lock_mode X locks gap before rec insert intention waiting
```

#### 4.5 加锁规则

主键索引：

- 等值条件，命中，记录锁
- 等值条件，未命中，间隙锁
- 范围条件，命中，包含where条件的临键区间，临键锁
- 范围条件，未命中，间隙锁

辅助索引：

- 等值条件，命中，命中记录的辅助索引项+主键索引项加记录锁，辅助索引两侧加间隙锁
- 等值条件，未命中，加间隙锁
- 范围条件，命中，包含where条件的临键区间加临键锁，命中记录的id索引加记录锁
- 范围条件，未命中，加间隙锁

#### 4.6 意向锁

InnoDB实现了表级锁，就是意向锁（Intention Lock）。意向锁是MySQL内部使用的，不需要用户干预，意向锁和行锁可以共存。

意向锁主要作用是为了全表更新数据时提升性能。当某一行有行写锁，全表进行更新，如果没有意向锁，则需要全表遍历每一行，查看是否有行级锁。

意向锁和读写锁的兼容关系

|      | IS   | IX   | S    | X    |
| ---- | ---- | ---- | ---- | ---- |
| IS   | Y    | Y    | Y    | N    |
| IX   | Y    | Y    | N    | N    |
| S    | Y    | N    | Y    | N    |
| X    | N    | N    | N    | N    |

- 意向锁相互兼容：IX、IS只是表明申请更低层级别的X、S操作
- 表级S锁和X、IX锁不兼容：加了表级S锁，不允许其他事务再加X锁
- 表级X锁和IS、IX、S、X不兼容：加了表级X锁，会修改数据

### 5. 死锁

多个session加锁顺序不一致，会造成死锁

避免死锁：

- 注意程序的逻辑
- 保持事务的轻量
- 提供运行速度
- 尽量快提交事务，减少持有锁的时间