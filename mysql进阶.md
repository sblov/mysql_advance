## Mysql进阶

### 编码设置

#### 服务器编码

> `show variables like 'character%'`	

​	查询所有跟字符集有关变量(session会话变量)

```shell
+--------------------------+--------------------------+
| Variable_name            | Value                    |
+--------------------------+--------------------------+
| character_set_client     | utf8                     |
| character_set_connection | utf8                     |
| character_set_database   | utf8                     |
| character_set_filesystem | binary                   |
| character_set_results    | utf8                     |
| character_set_server     | utf8                     |
| character_set_system     | utf8                     |
| character_sets_dir       | F:\mysql\share\charsets\ |
+--------------------------+--------------------------+
```

> `set names '字符集'`  

​	设置与client有关的字符集变量：`character_set_client` 、`character_set_connection`、`character_set_results`

> 修改 `my.ini` 文件

#### 数据库表编码

> `shwo create table 表名 `	

​	查看建表语句，来查看表编码

```sql
CREATE TABLE `表名` (
  `id` varchar(32) NOT NULL,
  ........
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

> `alter table 表名 character set 字符集`	

​	更改表字符集编码

#### 数据列编码

> `alter table  表名  change  原列名  修改后列名  数据类型  character  set  字符集 约束`

​	修改单个字段列的编码

e.g:	`alter table emp change emp_name empName varchar(50) charcater set utf8 not null`

> `alter table 表名 convert to character set 字符集;  `

​	修改表中所有字段列的编码

#### 修改多张表的编码

​	对于多张表编码的修改，都是以所在数据库为整体修改

1、导出所有表结构

> `mysqldump -uroot -p --default-character-set=字符集 -d 数据库名  > 路径/建表.sql`

2、将数据库的数据导出

> `mysqldump -uroot -p --quick  --no-create-info --extended-insert --default-character-set=字符集 数据库名 > 路径/数据.sql`

3、删除原有数据库

> `drop database 数据库`

4、重新以新的编码格式创建数据库

> 建数据库：`create database 数据库名 default charset 字符集`
>
> 导入建表：`mysql -uroot -p 数据库名 < 路径/建表.sql`
>
> 修改数据：在数据.sql加入	`set names '字符集'`，让插入数据都以该字符集编码插入
>
> 导入数据：`mysql -uroot -p 数据库名 < 路径/数据.sql`

### 变量

#### 会话变量

> `show session variables `  

​	查看所有会话级变量

> `show session variables like 'auto%' `  | `show variables like 'auto%'`

​	模糊查询

> `select @@session.变量名`

​	查询

> `set 变量名='值'`	| `set @@session.变量名='值'`

​	设置指定会话变量值

#### 全局变量

> `show global variables `  

​	查看所有全局变量

> `show global variables like 'auto%' ` 

​	模糊查询

> `select @@global.变量名`

​	查询

> `set global 变量名='值' `	| `set @@global .变量名='值'`

​	设置指定全局变量值

#### 局部变量

​	用于存储过程，作用范围在begin到end语句块之间 

> `declare 变量名  数据类型 default 默认值`

​	声明变量

> `select 变量名='值'`

​	设值

#### 用户变量 

> `set @变量名='值'`

​	变量设置

### 存储过程

#### 创建

1、选中数据库

2、改变分隔符

​	默认为   	`;`，通过	`delimiter 分隔符` 修改分隔符

```shell
delimiter $
```

3、存储语句

```sql
create procedure 存储名()
begin
	select version();
	.............
end
$
```

4、还原分隔符

5、调用存储过程

```shell
call 存储名；
```

#### 参数

**in（输入参数）**

​	不会修改传入的变量的值，而是将原变量的值赋值给该in参数

```shell
mysql> set @p = 1;  #设置用户变量
mysql> delimiter $
mysql> create procedure lov(in p_1 int )	#in参数
    -> begin
    -> set p_1 = 2;
    -> select p_1;
    -> end
    -> $
mysql> delimiter ;
mysql> call lov(@p);	#存储过程调用
+------+
| p_1  |
+------+
|    2 |
+------+
mysql> select @p;	#原始用户变量
+------+
| @p   |
+------+
|    1 |
+------+
```

**out（输出参数）**

​	对于传入的变量，都会重置为null，参数的值会影响原变量

```shell
mysql> create procedure lov(out p_o int)
    -> begin
    -> select p_o;
    -> end
    -> $
mysql> set @p=1;
mysql> call lov(@p);
+------+
| p_o  |
+------+
| NULL |
+------+
mysql> select @p$
+------+
| @p   |
+------+
| NULL |
+------+
```

**inout（输入输出参数）**

​	in与out的结合，相当于函数中按址传参

```shell
mysql> create procedure lov(inout p_io int)
    -> begin
    -> select p_io;
    -> set p_io=2;
    -> end
    -> $
mysql> set @p=1;
mysql> call lov(@p);
+------+
| p_o  |
+------+
|  1   |
+------+
mysql> select @p$
+------+
| @p   |
+------+
|  2   |
+------+
```

#### 流程控制

> `if ... else  ...  end if; `      ,    `if ...then ...elseif ... then...else ... end if;`

>`case ... when ... then... else ...end case;`

> `while ... do ... end while;`

>`repeat ... until ... end repeat;`

>`loop_name : loop ... if ... then leave loop_name; end if; end loop;`

```sql
create procedure lov()
begin 
	declare uname VARCHAR(50) default 'lov';
	declare age INT default 20;
	declare salary int default 2000;
	WHILE age < 40 do 
		set age = age + 1;
		set salary = salary +111;
		case salary 
			when 2111
			then 
				set salary = salary *2;
				insert into user (name,age,salary) VALUE (uname,age,salary);
				set salary = salary/2;
			else 
				set uname = 'lov2';
				insert into user (name,age,salary) VALUE (uname,age,salary);
		end case; 
		
	end while;
end
```

#### 定义条件与处理

​	条件定义和处理用于定义在过程执行中遇到问题时的相应处理

> `declare continue handler for sqlstate '错误代码' set 变量=变量值`

```shell
mysql> create procedure lov()
    -> begin
    -> insert into user value (61,'lov',20,20000);
    -> insert into user value (1,'lov',20,20000);
    -> end
    -> $
Query OK, 0 rows affected (0.00 sec)
mysql> call lov()$
ERROR 1062 (23000): Duplicate entry '61' for key 'PRIMARY' #23000为错误代码
---------------------------------------------------------------------------
#处理错误代码行，并继续执行之后的语句
mysql> create procedure lov()
    -> begin
    -> declare continue handler for sqlstate '23000' set @x = 23000； 
    -> insert into user value (61,'lov',20,20000);
    -> insert into user value (2,'lov',20,20000);
    -> end
    -> $
mysql> call lov()$
Query OK, 1 row affected (0.05 sec)
```

#### 管理

> `show procedure status where db='数据库名'`

​	显示该数据库中所有存储过程信息

> `select specific_name from mysql.proc`   --where specific_name = '存储过程名'

​	选出现有存储过程名 （具体查询）

> `select 存储过程名,body from mysql.proc`   |   `show create procedure 存储过程名`

​	查询存储过程内容

> `drop procedure  存储过程名 `  |   `drop procedure if exists 存储过程名`

​	删除存储过程

> `alter procedure ........`

​	修改具体参数，参照mysql官方文档

### 函数

#### 创建

1、查看是否开启创建函数功能

```shell
mysql> show variables like '%fun%';	
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin_trust_function_creators | OFF   |		#为开启创建函数功能
+---------------------------------+-------+
mysql> set global  log_bin_trust_function_creators = 1; #开启
```

2、创建函数语法

```shell
create function 函数名 (变量,...)
returns 数据类型
begin 
	......
	return 数据;
end;
---------------------------------------------demo
mysql> delimiter $
mysql> create function fun_add(a int,b int)
    -> returns int
    -> begin
    -> return a+b;
    -> end;
    -> $
```

#### 管理

> `show create function 方法名`

> `eeeedrop function  if exists 方法名`

### 视图

​	由查询结果形成的一张虚拟表

#### 优势

> 简化查询语句

> 进行权限控制

> 分表

#### 创建

```sql
CREATE
    [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    --Merge：合并的执行方式，每当执行的时候，先将视图的sql语句与外部查询视图的sql语句，混合到一起，最终执行
    --TempTable：临时表模式，每当查询的时候，将视图所使用的select语句生成一个结果的临时表，再在当前的临时表内进行查询
    --undefined：mysql将选择所要使用的算法，但一般倾向于merge，因为merge更有效，如果使用临时表，视图则不可更新
    [DEFINER = { user | CURRENT_USER }]
    [SQL SECURITY { DEFINER | INVOKER }]
    VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION]
    --with check option:更新视图的数据时，必须满足视图的条件，满足之后才能更新到基表中：如视图时查询基表某范围的值，而对视图的更新，只有更新该范围的值才会对基表中的数据进行更新，否则不会对基表中数据产生影响
```

```sql
create view v_dept as select * from department;
```

#### 管理

> ` select * from information_schema.views where TABLE_NAME = '视图名'\G;`

​	视图都存放在information_schema数据库的views表里，通过该sql查询该视图的具体内容，或在视图所在数据库下通过`show tables` 查询是否包含该视图

```shell
       TABLE_CATALOG: def
        TABLE_SCHEMA: lov
          TABLE_NAME: v_dept
     VIEW_DEFINITION: select `lov`.`department`.`id` AS `id`,`lov`.`department`.`department_name` AS `department_name` from `lov`.`department`
        CHECK_OPTION: NONE
        IS_UPDATABLE: YES
             DEFINER: root@localhost
       SECURITY_TYPE: DEFINER
CHARACTER_SET_CLIENT: utf8
COLLATION_CONNECTION: utf8_general_ci
```

> `show table status [from 视图名称][like '匹配']`	

​	查看视图的定义（[]为可选）

> `select drop_priv from mysql.user where user='root'`

​	查看是否有删除权限

> `drop view if exists 试图名`

​	删除视图，多个视图直接    `,`分割

#### 视图更新

​	某些视图是可更新的，可以在update、delete、inert等语句中使用，以更新基表的内容。对于可更新的视图，在视图的行和基表中的行之间必须具有一对一的关系。

​	还有一特定的其他结构，这类结构会使视图不可更新：

```sql
聚合函数（SUM(),MIN()...)
DISTINCT
GROUP BY
HAVING
UNION
位于选择列表中的子查询
JOIN
FROM 子句中的不可更新视图
WHERE子句中的子查询，引用FROM子句中的表
仅引用文字值
ALGORITHM = TEMPTABLE
```

### 触发器

​	触发器是一种特殊的存储过程，它在插入，删除或修改特定表中的数据时触发执行，它比数据库本身标准的功能有更精细和更复杂的数据控制能力

​	触发器有数据库主动去执行，不能被直接调用

> 监视地点：一般为表名

> 监视事件：update/dalete/insert

> 触发时间：after/before

> 触发事件：update/delete/insert

#### 创建

```sql
CREATE
    [DEFINER = { user | CURRENT_USER }]
    TRIGGER trigger_name
    trigger_time trigger_event
    ON tbl_name FOR EACH ROW
    [trigger_order]
    trigger_body

--trigger_time: { BEFORE | AFTER }

--trigger_event: { INSERT | UPDATE | DELETE }
--对于insert：新插入的行可以用new来表示，行中的某列可以通过‘new.列名’ 来表示
--对于delete:被删除的一行，可以用old来引用，‘old.列名’ 表示某列
--对于update：update前的数据用old，update后的数据用new

--trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

INSERT：

```sql
CREATE TRIGGER tr_insertDept AFTER INSERT ON department FOR EACH ROW
BEGIN
	INSERT test ( dept_id, COMMIT )
VALUES
	( new.id, 'new add' );
END;
```

UPDATE：

```sql
CREATE TRIGGER tr_updateDept AFTER UPDATE ON department FOR EACH ROW
BEGIN
	UPDATE test 
	SET COMMIT = new.department_name 
WHERE
	dept_id = old.id;
END;
```

#### 管理

> `show triggers`

​	查看当前数据库中所有触发器

> `desc information_schema.TRIGGERS`

​	information_schema.TRIGGERS表中，存储所有库中的触发器

> `select * from information_schema.TRIGGERS where TRIGGER_NAME='trigger_name' `

​	查询触发器

> `drop trigger [schema_name.]trigger_name`

​	删除

### My ISAM表锁

​	**mysql锁的机制比较简单，其最显著的特点就是不同的存储引擎支持不同的锁机制。如：myISAM和MEMORY存储引擎采用的是表级锁；BDB存储引擎采用的页面锁，但也支持表级锁；InnoDB存储引擎既支持行级锁，也支持表级锁，但默认情况采用行级锁**

​	MySQL三种锁的特效：开销、加锁速度、死锁、粒度、并发性能

> **表级锁：**开销小、加锁快；不会出现死锁；锁定粒度大，发生冲突的概率最高、并发度最低
>
> **行级锁：**开销大、加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低、并发度也最高
>
> **页面锁：** 开销和加锁时间介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁与行锁之间、并发度一般

​	仅从锁的角度来看：

​	***表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用（web应用）***

​	***行级锁则更适合于大量按索引条件并发更新少量不同数据，同时又有并发查询的应用***

#### 表锁

​	mysql表级锁有两种模式：表共享读锁（table read lock）和表独占写锁（table write lock）

​	==确保该表使用My ISAM存储引擎==

> `lock table 表名 read `   

​	加共享读锁

> `lock table 表名 write`

​	加表独占写锁

> 追加   `,表名 read/write`  对多个表加锁

*一个客户端加共享读锁，另一个客户端只能读取，不能修改（进入锁等待），需要当地该表的锁被释放（unlock）*

![1548739643524](img\1548739643524.png)

> 一个session使用LOCK TABLE命令给表A加了锁，这个session可以查询锁定表中的记录，但更新或访问其他表就会出现错误；
>
> `ERROR 1100 (HY000): Table '....' was not locked with LOCK TABLES`

#### 并发插入

​	My ISAM表的读和写是串行的，但这个是总体而言；在一定条件下，My ISAM表也支持查询和插入的并发执行

​	My ISAM存储引擎有一个系统变量concurrent_insert，专门用来控制其并发插入的行为，其值为0、1、2

> **0：**不允许并发插入
>
> **1：**如果My ISAM表中间没有被删除的行（空洞），My ISAM允许在一个进程读表的同时，另一个进程可以从表尾插入记录。mysql默认设置
>
> **2：**无论My ISAM表中有没有空洞，都允许在表尾并发插入

```shell
mysql> show variables like 'concurrent_insert';#查询变量
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| concurrent_insert | AUTO  |
+-------------------+-------+
mysql> set global concurrent_insert = 2; #变量更改
mysql> lock table department read local; #给当前进程加锁
mysql> insert into department(department_name) values ('adc');
ERROR 1099 (HY000): Table 'department' was locked with a READ lock and can't be updated  # error
#########另一个进程insert
mysql> insert into department(department_name) values ('adc');
Query OK, 1 row affected (0.00 sec)

mysql> select * from department;
+----+-----------------+
| id | department_name |
+----+-----------------+
|  1 | AA1             |
|  2 | BB              |
|  3 | DDD             |
|  4 | QQQ             |
|  8 | XXX             |
|  9 | adc             |
+----+-----------------+
#############该进程再查询
mysql> select * from department;
+----+-----------------+
| id | department_name |
+----+-----------------+
|  1 | AA1             |
|  2 | BB              |
|  3 | DDD             |
|  4 | QQQ             |
|  8 | XXX             |
+----+-----------------+
```

#### 写优于读

> ​	当一个进程请求某My ISAM表的读锁，同时另一个进程也请求同一表的写锁，无论那个先请求，都是**写进程先获得锁**。
>
> ​	mysql中写请求比读更重要，My ISAM不适合有大量更新与查询操作，因为大量的更新会造成查询很难获得锁，从而一直阻塞

​	*可通过  `set low_priority_updates = 1`，使该连接发出的更新请求优先级降低，对于insert，delete都可以通过该方式指定*

### 事务

> `shwo engines;`

​	查询数据库下的存储引擎的支持情况

```shell
mysql> show engines \G;
*************************** 1. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES  # 只有InnoDB支持事务
          XA: YES
  Savepoints: YES
```

> `show variables like '%storage_engine%';`

​	查看mysql当前默认存储引擎

> `show create table table_name`

​	通过查看表创建语句查看表的存储引擎

> `start transaction;`

​	开启事务

> `commit`

​	事务提交

> `set autocommit = 0`

​	关闭事务自动提交，既开启事务后，没有commit，事务不会提交执行

```shell
mysql> select @@session.autocommit;
+----------------------+
| @@session.autocommit |  #默认开启
+----------------------+
|                    1 |
+----------------------+
```

> `commit and chain`

​	表示提交事务后重新开启新的事务

> `rollback and release`

​	表示事务回滚之后断开和客户端连接

> `savepoint point_name`

​	可以通过rollback to point_name回滚到指定状态

```shell
mysql> start transaction;

mysql> savepoint s1;

mysql> insert department(department_name) values('UU');

mysql> select * from department;
+----+-----------------+
| id | department_name |
+----+-----------------+
| 13 | UU              |
+----+-----------------+

mysql> rollback to s1;  #回到s1点，在此commit会只执行到s1处
mysql> select  * from department; 
Empty set (0.00 sec)
```

***对于InnoDB锁，在加锁后，执行start transaction会造成一个隐式的unlock tables的执行***

### 慢查询

​	mysql记录下查询超过指定时间的语句，将超过指定时间的sql语句查询称为慢查询

> `show variables where variable_name = 'long_query_time'`

​	查询最长查询时间

> `show status like 'uptime'`

​	当期数据库运行时间

> `show status like 'com_Select'`

​	select执行次数

> `show status like 'connections'`

​	连接数

> `show global status like 'slow_queries'`

​	慢查询数

> `mysqld.exe --safe-mode --slow-query-log`

​	开启安全模式和慢查询日志，日志保存位置为datadir配置的目录

> `set @@global.slow_query_log=on`

​	进入数据库后开启慢查询日志，开启后在对应目录会生成初始化log文件

```shell
mysql> set @@global.slow_query_log=on;

mysql> show variables like 'slow%';
+---------------------+--------------------------------------------+
| Variable_name       | Value                                      |
+---------------------+--------------------------------------------+
| slow_launch_time    | 2                                          |
| slow_query_log      | ON                                         |
| slow_query_log_file | D:\mysql-5.7\data\PC-201812290617-slow.log |
+---------------------+--------------------------------------------+
```

#### 配置

​	在配置文件中(my.ini)

```shell
slow-query-log=1  
long_query_time = 5 
slow-query-log-file=slow.log #系统默认为datadir配置的目录下的host_name-slow.log
log-queries-not-using-indexes #没有使用索引的sql也会记录日志
```

​	***修改配置后重启服务***

### 索引

​	在关系型数据库中，索引是一种与表有关的数据结构，可以使对应表的sql语句执行更快

#### 分类

**主键索引**

​	主键即为主键索引

> `alter table 表名 add PRIMARY KEY  列名`

​	添加主键索引

**唯一索引**

​	唯一索引所在的列可以为null值；不能为空字符串

> `create unique index 索引名称 on 表名(列名)`

​	创建唯一索引

​	**通过在sql语句前添加 `explain` ，能得到该sql执行的具体效率等信息**

```shell
mysql> explain select id from user where id = 73 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using index
```

**全文索引**

​	FULLTEXT索引，用于全文索引。只有MyISAM和InnoDB表类型支持FULLTEXT索引。只可以从char，varchar和text列中创建

```sql
CREATE TABLE articles (
          id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
          title VARCHAR(200),
          body TEXT,
          FULLTEXT (title,body)  --FULLTEXT index
        ) ENGINE=InnoDB;
--use index
SELECT * FROM articles
        WHERE MATCH (title,body)
        AGAINST ('database');
```

**普通索引**

> `create index 索引名称 on 表名(列名)`

​	创建普通索引

#### 管理

> `show keys from table_name`  	| 	`show index from table_name`

​	查询索引

> `alter table 表名 drop index 索引名称 `

​	删除索引

#### 原理

​	通过二叉树对索引文件进行检索

​	在创建表时，会创建三个文件（对应datadir配置的文件下）

> **.frm ：**表示表的结构
>
> **.myd**：表示数据
>
> **.myi**：表示索引的文件，该文件在移动目录后会失效，而以上两个不会

![](assets\1548856383924.png)

​	**索引会导致insert、update、delete的效率**

#### 应用

> 1、快速查找符合where条件的记录（或查询的所以字段为索引字段）
>
> 2、快速确定候选集，若where条件使用了多个索引字段，则mysql会优先使用能使候选集规模最小的那个索引，以便尽快排除不符合条件的记录
>
> 3、如果表中存在几个字段构成的联合索引，则查找记录时，这个联合索引的最左前缀匹配字段也会被自动作为索引来加入查找
>
> ​	如：（a,b,c）构成联合索引，则（a，b）（a，b，c）（a）均作为索引
>
> 4、多表做join操作时会使用索引（如果参加join的字段在这些表中均建立了索引的话）
>
> 5、若某字段已建立索引，求该字段的max与min时会使用索引
>
> 6、对建立了索引的字段锁sort或group操作时，会使用索引

#### Profile分析

> `select @@have_profiling`

​	查看是否支持	    `show profile`

> `set profiling = 1`

​	开启session级别profiling

> `show profiles`

​	查询queryID

> `show profile for query queryID`

​	查看执行过程

```shell
mysql> set profiling =1;
mysql> select * from user;
mysql> show profiles;
+----------+------------+--------------------+
| Query_ID | Duration   | Query              |
+----------+------------+--------------------+
|        1 | 0.00029350 | select * from user |
+----------+------------+--------------------+
mysql> show profile for query 1; #根据每个阶段的运行时间选择优化
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000054 |
| checking permissions | 0.000007 |
| Opening tables       | 0.000020 |
| init                 | 0.000016 |
| System lock          | 0.000007 |
| optimizing           | 0.000003 |
| statistics           | 0.000011 |
| preparing            | 0.000010 |
| executing            | 0.000002 |
| Sending data         | 0.000078 |
| end                  | 0.000004 |
| query end            | 0.000005 |
| closing tables       | 0.000006 |
| freeing items        | 0.000059 |
| cleaning up          | 0.000013 |
+----------------------+----------+
```

### mysql优化

#### 表优化

> `analyze  [local | no_write_to_binlog ] table tb_name [,tbale_name] `

​	分析表

> `check table tb_name [,tb_name] [option]`

​	检查表

​	`check table`也可以检查视图是否有误，比如在视图定义中引用的表已不存在

> `optimize [local | no_write_to_binlog] table tb_name [,tb_name]`

​	优化表

​	`optimize table` 只对MyISAM，BDB，InnoDB表起作用

​	**对于MyISAM表，`optimize table`按如下方式操作：**

> 如果表已经被删除或分解了行，则修复表
>
> 如果未对索引页进行分类，则进行分类
>
> 如果表的统计数据没有更新（并且通过对索引进行分类不能实现修复），则进行更新

==***以上的操作在执行时，都会对表进行锁定***==

> `show table status [like '...'] `

​	查询表状态信息

```shell
mysql> show table status \G;
*************************** 1. row ***************************
           Name: user		#表名称
         Engine: InnoDB		#存储引擎
        Version: 10			#版本
     Row_format: Dynamic	#行格式
           Rows: 20			#表中行数
 Avg_row_length: 819		#平均每行包括的字节数
    Data_length: 16384		#整个表的数据量（byte）
Max_data_length: 0			#表可以容纳的最大数据量
   Index_length: 0			#索引占用磁盘的空间大小
      Data_free: 0			#对于MyISAM引擎，标识已分配，但未使用及已被删除行的空间
 Auto_increment: 82			#下一个主键值
    Create_time: 2019-01-27 11:14:53		#表的创建时间
    Update_time: NULL		#表最近更新时间
     Check_time: NULL		#使用check table或myisamchk工具检查表的最近时间
      Collation: utf8_general_ci		#表的默认字符集和字符排序规则
       Checksum: NULL		#如果开启，则对整个表的内容计算时的校验和
 Create_options:			#表创建时的其他选项
        Comment:			#注释信息
```

#### 表分区

​	当数据量过大时，需要将一张表的数据划分几张表存储。一些查询可以得到极大的优化，这主要借助于满足一个给定where语句的数据可以中保持在一个或多个分区内，这样在查找时就不用查找其他剩余的分区

> `show plugins`

​	是否支持分区，查看当中的partition是否为active

##### 分区创建

**Range分区**

```sql
partition by range (字段名)( --如果有主键，该字段名必须包含主键
	partition p1 values less than (11),	--该字段值小于11，为p1分区
    partition p2 values less than (21),
	partition p3 values less than (31)
)	--在insert数据时，不能超过该分区的数值规定
```

**List分区**

```sql
partition by list (字段名)( 
	partition p1 values in (1,3,5,7,9),	
    partition p2 values less than (2,4,6,8,10),
	partition p3 values less than (11,13,15,17,18)
)
```

**Hash分区**

```sql
partition by hash(字段名)(
	partition 4;  	--分成四个区，通过hash得到分区编号；不写则默认为一个分区
)
```

**Keys分区**

```sql
partition by linear key(字段名)(
	partition 4;  	--分区编号通过2的幂得到；不写则默认为一个分区
)
```

##### 分区管理

> `alter table tb_name drop partition prt_name`

​	删除分区，对应分区的内容也会删除

> `alter table tb_name add partition(partition prt_name value less than | in .....)`

​	为range或list分区增加分区

#### 内存优化

#### 应用程序优化

#### 账号权限

#### 监控

#### 定时维护

#### 备份还原

---

## Navicat for MySQL快捷键

>1. **ctrl+q            打开查询窗口**  
>2. **ctrl+/            注释sql语句**  
>3. **ctrl+shift +/     解除注释**  
>4. **ctrl+r            运行查询窗口的sql语句**  
>5. **ctrl+shift+r      只运行选中的sql语句**  
>6. **F6                打开一个mysql命令行窗口**  
>7. **ctrl+d           （1）：查看表结构详情，包括索引 触发器，存储过程，外键，唯一键;（2）：复制一行**  
>8. **ctrl+l            删除一行**  
>9. **ctrl+n            打开一个新的查询窗口**  
>10. **ctrl+w            关闭一个查询窗口**  
>11. **ctrl+tab          多窗口切换** 



