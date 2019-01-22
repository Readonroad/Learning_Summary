### 数据库定义
数据库是物理操作文件系统或其他形式文件类型的集合。

实例：MySQL数据库由后台线程以及一个共享内存区组成

MySQL中，数据库和实例是一一对应的，无法直接操作数据库，需要通过数据库实例来操作数据库文件。数据库实例可以看做是数据库为上层提供的一个专门用于操作的接口。

### 数据库模型
层次模型：类似于一棵树，通过“上下级”的层次关系组织数据

网状模型：类似于城市之间的网络，把每个数据节点和其他节点连接起来

关系模型：二维表格，通过行+列唯一确定数据，更加简单，实用跟多。

### SQL
SQL(Structured Query Language)用来访问和操作数据库系统，SQL语句既可以查询数据库中的数据，也可以添加、更新和删除数据库中的数据，对数据库进行管理和维护操作。

### MySQL
MySQL是目前应用最广泛的开源关系数据库，与其他数据库的差别在于，MySQL实际上只是一个SQL接口，它的内部含有多种数据引擎，常用的包括：InnoDB,MyISAM等。
* MySQL接口和数据库引擎的关系类似浏览器和浏览器引擎的关系，对用户而言，切换浏览器引擎不影响浏览器界面，切换MySQL数据库引擎不影响自己写的应用程序使用MySQL的接口。
* 使用MySQL时，不同的表还可以使用不同的数据库引擎。如果你不知道应该采用哪种引擎，记住总是选择InnoDB就好了。
* InnoDB存储引擎中，所有的数据都被逻辑地存放在表空间中，表空间是存储引擎中最高的存储逻辑单位，表空间下面又包括segment(段)、extent(区)、page(页)
### MySQL使用
SQL语言关键字不区分大小写，不同的数据库，表名和列名，有些区分，有些不区分。可以自己统一规则。
#### 登录MySQL和查看MySQL基本信息
```sql
mysql -D 所选择的数据库名 -h 主机名 -u 用户名 -p 
mysql> exit;  #退出（quit)
mysql> status;  # 显示当前mysql的version的各种信息
mysql> select version(); # 显示当前mysql的version信息
mysql> show global variables like 'port'; # 查看MySQL端口号
```
#### 创建数据库
数据库由一个或数个表格组成，对表操作需要先进入库：use 库名

数据库管理相关命令
```sql
#character set name 指定字符集为name;
create database db_name character set gbk; #创建一个名为db_name的数据库，且编码指定为gbk
drop database db_name;   #删除数据库
show databases;          #显示数据库列表
use db_name;             #进入数据库db_name
show tables;             #显示数据库db_name下所有表名字
describe table_name;     #显示表tablename的结构
delete from table_name;  #清空表table_name中的记录，表依然存在
drop table table_name;   #删除数据表table_name,需要重建
```
创建一个数据表：create table 表名（列声明）；
```sql
DROP TABLE IF EXISTS my_table; 
CREATE TABLE my_table(
    'id'    INT(100) NOT NULL AUTO_INCREMENT primary key,
    'name'  VARCAHR(32) NOT NULL DEFAULT '' COMMENT '名称',
    'gender'  VARCAHR(32) NOT NULL DEFAULT '' COMMENT '性别',
    'age'   TINYINT(32)     NOT NULL DEFAULT 0  COMMENT '年龄',
    'create_at' DATETIME    NOT NULL,
    --创建唯一索引
    UNIQUE INDEX idx_usr_name('name')
)
-- 指定数据引擎为InnoDB,默认的字符集为utf-8
ENGINE = InnoDB DEFAULT CHARSET=utf-8\
COMMENT= '对表或字段进行说明'
```
#### 数据表操作：增删改查
#### SELECT
SELECT语句用于从表中选取数据。

基本用法：
```
SELECT * FROM 表名;  查询表
```
条件查询
```
SELECT * FROM 表名 (表别名) WHERE 条件表达式;  查询表 ,where指定查询条件
```
```sql
#条件表达式：OR / ADN/ <>(不相等) / LIKE'%AB%'(含有‘AB',%表示任意字符)
SELECT * FROM my_table WHERE age>10 and gender='M';
SELECT * FROM my_table WHERE age=10 LIMIT 3 OFFSET 2;
```
投影查询
```
SELECT 列名称1 (别名), 列名称2 (别名2) FROM 表名 (表别名) WHERE 条件表达式; 查询表中指定列的信息
```
```sql
SELECT id,name,age FROM my_table WHERE age>10 ORDER BY age ASC;
```
数据排序查询:ORDER BY
```sql
#默认排序方式为升序ASC,降序为DESC
SELECT * FROM my_table WHERE age>10 ORDER BY age ASC;
SELECT * FROM my_table WHERE age>10 ORDER BY age ASC HAVING 条件表达式;#having 用来过滤分组
```
分页查询：LIMIT num OFFSET start;从结果集中start开始，最多截取num个;
```sql
SELECT * FROM my_table WHERE age>10 ORDER BY age ASC LIMIT 3 OFFSET 3;
```
聚合查询：方便对数据表中某个信息进行统计
```sql
SELECT COUNT(*) 别名 FROM 表名 WHERE 条件表达式;    #统计满足条件的个数，且指定该显示的别名
SELECT 列名1, 列名2, COUNT(*) 别名 FROM 表名 GROUP BY 列名1, 列名2; #根据某列分组，统计每个分组的结果
--其他函数：求和SUM(),平均值AVG(), 最大值MAX(),最小值MIN(),其中SUM和AVG必须是数值类型，MAX和MIN也可以是字符型，返回排序最后和排序最前的字符
```
多表查询：从多张表中同时查询数据，显示的结果是多个表的“乘积”
```sql
SELECT * FROM 表1,表2,... WHERE 条件表达式;
#设置列的别名和表的别名
SELECT 表1.列名1 别名11, 表1.列名2 别名12, ...,
       表2.列名1 别名21, 表2.列名2 别名22,... 
       FROM 表1,表2 WHERE 条件表达式;
SELECT 别名表1.列名1 别名11, 别名表1.列名2 别名12, ...,
       别名表2.列名1 别名21, 别名表2.列名2 别名22,... 
       FROM 表1 别名表1, 表2 别名表2 WHERE 条件表达式;
--注:多表查询时，可以多个表乘积的结果，导致查询的结果集会很大
```
连接查询：对多个表进行JOIN运算，即先确定一个主表作为结果集,然后，把其他表的行有选择性“连接”在主表结果集上
```sql
#内连接:INNER JOIN 表名 表别名 ON 条件 (WHERE、ORDER BY)
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s INNER JOIN classes c ON s.class_id = c.id;

#外连接：RIGHT OUTER JOIN 表名 表别名 ON 条件
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s RIGHT OUTER JOIN classes c ON s.class_id = c.id;

SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s LEFT OUTER JOIN classes c ON s.class_id = c.id;

SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s FULL OUTER JOIN classes c ON s.class_id = c.id;
-- INNER JOIN:两张表记录的交集; RIGHT OUTER JOIN:选出右表中存在的记录
-- LEFT OUTER JOIN:选出左表中存在的记录; FULL OUTER JOIN:左右表记录的并集
-- 注：如果有些记录只存在一张表中，那么结果集中就会用NULL填充对方不存在的记录
```
#### INSERT
向数据表中插入一条新的记录，基本语法为：
```sql
#插入一条指定字段的记录
INSERT INTO 表名 (字段1,字段2,...) VALUES (值1,值2,...)
INSERT INTO 表名 VALUES (值1,值2,...)
#将一个表的数据插入另一个表
INSERT INTO 表1 (字段1,字段2,...) SELECT 表2.字段1,表2.字段2,... FROM 表2 WHERE 条件表达式;
```
#### UPDATE
更新数据表中的数据
```sql
UPDATE 表名 SET 字段1 = 值1,字段2 = 值2,... WHERE 条件表达式;
UPDATE 表名 SET 字段1 = 值1,字段2 = 值2,...; #更新整个表中所有的记录
--当有匹配的记录时，返回更新成功，且显示更新的行数和WHERE语句匹配的行数
--如果没有匹配记录，也返回成功，显示更新行数和WHERE语句匹配行数为0
```
#### DELETE
删除数据表中的记录
```sql
DELETE FROM 表名; #删除数据表中的记录，单表依然存在
DELETE FROM 表名 WHERE 表达式; #根据条件删除记录
--当有匹配的记录时，返回成功，且显示删除的行数和WHERE语句匹配的行数
--如果没有匹配记录，也返回成功，显示删除行数和WHERE语句匹配行数为0
```
### 数据表添加索引
#### 主键索引
ALTER TABLE 表名 ADD PRIMARY KEY (字段名);

#### 唯一索引
ALTER TABLE 表名 ADD UNIQUE (字段名);

#### 多列索引
ALTER TABLE 表名 ADD INDEX index_name (字段1,字段2,...);
### 数据表结构的修改
#### 为数据表添加一列
ALTER TABLE 表名 ADD 列名 列数据类型 [after 插入位置];
```sql
-- 在名为 age 的列后插入列 birthday: 
ALTER TABLE students ADD birthday date after age;
-- 在名为 number_people 的列后插入列 weeks: 
ALTER TABLE students ADD column `weeks` varchar(5) not null default "" after `number_people`;
```
#### 修改数据表的列信息
ALTER TALBE 表名 CHANGE 列名 列新名字 新数据类型;
#### 删除数据表中的某一列
ALTER TALBE 表名 DROP 列名;
#### 重命名数据表
ALTER TALBE 表名 RENAME 新表名；

### 事务和隔离级别
事务：满足ACID特性的一组操作称为事务，可以通过commit提交一个事务，也可以通过rollback回滚，回到执行前的状态。

* A:atomic 原子性，不可分割的最小单元，所有操作要不全部执行成功，要不全部失败;
* C:consistency 一致性，事务完成后，所有数据状态都是一致的。对一个数据的读取结果都是相同的;
* I:isolation 隔离性，并发时，一个事务的提交必须与其他事务相互隔离;
* D:durability 持久性,事务提交，所有的修改都会永远保存在数据库中，即使数据库崩溃，执行结果也不会丢失。

默认情况下，每一条SQL语句都被当做一个隐式的事务自动提交执行。
#### 显示事务
```sql
BEGIN;     #BEGIN 开始一个事务
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;    #COMMIT 提交一个事务
--即试图把事务内的所有SQL所做的修改永久保存。如果COMMIT语句执行失败了，整个事务也会失败。
```
```sql
BEGIN;     #BEGIN 开始一个事务
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
ROLLBACK;    #ROLLBACK 回滚事务，整个事务失败。
--数据库事务是由数据库系统保证的，我们只需要根据业务逻辑使用它就可以。
```
#### 隔离级别
对于两个并发执行的事务，如果涉及操作同一条记录时，可能是发生问题，带来数据不一致。事务的隔离级别是提供给用户用于在性能和可靠性之间做出选择和权衡的配置项。四种隔离级别：READ UNCOMMITED、READ COMMITED、REPEATABLE READ 和 SERIALIZABLE；每个事务的隔离级别其实都比上一级多解决了一个问题：

* READ UNCOMMITED（未提交读）：一个事务读取到了另一个事务更新但未提交的数据，如果更新的事务失败回滚，那么当前事务读取的数据就是脏数据。
```
当两个事务并行时，session1第1次查询数据a后，session2对数据a进行了修改，session1再次查询数据a,即为修改后的数据，此时若sesssion2执行失败，则session1第2次查询的数据即为“脏数据”。导致这个问题的原因是：修改数据a时，未锁定。
```
* READ COMMITED（提交读）：一个事务内多次读取统一数据时，该事务还未结束，另一事务恰好对该数据进行了修改，导致第一个事务两次查询可能得到不同的结果
```
当两个事务并行时，session1第1次查询数据a,session2对数据a进行修改，因为有记录锁，因此解决了“脏读"的问题。session2提交后，session1读取数据a,得到的即为修改后的数据。导致一个事务session1前后查询的数据不一致。原因是：记录数据时，加了锁，解决了“脏读”的问题，但是查询数据未加锁，从而出现了“不可重复读”的问题。
```
* REPEATABLE READ（可重复读）：一个事务读取一条记录，发现没有，另一事务添加该记录并提交后，当前事务再次读取该记录时，依然不存在，但是对该记录进行更新时，却成功，好像之前的查询是幻觉，即为幻读。（一个事务多次查询同一个范围内的数据，而另一个事务对该范围内的数据添加了新的记录，当前事务会返回第一次查询的快照，不会返回不同的查询结果，可能发生幻读）
```
多次查询时，session2虽然添加了新数据，但是由于解决了“不可重复读”的问题，session1查询结果依然是上一次的查询结果。但是对新插入的数据进行更新时，却能成功，再次查询时，该数据就出现了。导致上一次的查询结果是幻读。
```
* SERIALIZABLE（可串行化）：最严格的隔离级别，所有事务按照次序依次执行，不会出现“脏读”“不可重复读”“幻读”的情况。由于所有事务都是串行执行，所以效率会大大下降，性能急剧下降。

注：数据库没有指定隔离级别时，会使用默认的隔离级别。MySQL中，如果使用InnoDB,默认隔离级别是REPEATABLE READ。

### 封锁粒度
解决并发操作数据不一致的问题，MySQL提供了两种封锁粒度：行级锁和表级锁。

应该尽量只锁定需要修改的那部分数据，而不是所有的资源。锁定的数据量越少，发生锁争用的可能就越小，系统的并发程度就越高。

但是加锁需要消耗资源，锁的各种操作（包括获取锁、释放锁、以及检查锁状态）都会增加系统开销。因此封锁粒度越小，系统开销就越大。

在选择封锁粒度时，需要在锁开销和并发程度之间做一个权衡。


