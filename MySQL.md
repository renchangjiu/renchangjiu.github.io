# MySQL语句和命令大全

## 一、用户连接、创建、权限、删除

### 1. 连接MySQL操作

```Bash
mysql -h 主机地址 -u 用户名 -P 端口号 -p
```

使用 SSL 连接

```Bash
mysql --ssl-ca=ca.pem --ssl-cert=client-cert.pem --ssl-key=client-key.pem -h主机地址 -u用户名 -p
```

### 2. 创建用户

```SQL
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

- `host` 指定该用户在哪个主机上可以登陆,如果是本地用户可用`localhost`, 如果想让该用户可以从任意远程主机登陆,可以使用通配符`%`.

### 3. 授权

```SQL
GRANT [all privileges/某个权限] ON databasename.tablename TO 'username'@'host';
```

如果想让该用户可以授权,用以下命令:

```SQL
GRANT all privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

### 4. 锁定用户

```SQL
ALTER USER 'username'@'host' ACCOUNT LOCK;
```

***解锁***

```SQL
ALTER USER 'username'@'host' ACCOUNT UNLOCK;
```

**常见场景**:

1 创建读写权限的用户

```SQL
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, EXECUTE, CREATE VIEW, SHOW VIEW ON databasename.tablename TO 'username'@'host';
```

2 创建只读权限用户

```SQL
GRANT SELECT,SHOW VIEW ON databasename.tablename TO 'username'@'host';
```

### 4. 设置与更改用户密码

***方法一***

```SQL
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

如果是当前登陆用户用

```SQL
SET PASSWORD = PASSWORD("newpassword");
```

***方法二***

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
```

### 5. 撤销用户权限

```SQL
REVOKE [ALL/某个权限] ON databasename.tablename FROM 'username'@'host';
```

某个用户权限可以用命令`SHOW GRANTS FOR 'username'@'host';` 查看.

### 6. 重命名用户

```SQL
rename user 'old_name'@'host' to 'new_name'@'host';
```

### 7. 删除用户

```SQL
DROP USER 'username'@'host';
```

### 8. 要求使用ssl登陆

```SQL
# 修改已存在用户  
ALTER USER 'username'@'%' REQUIRE SSL;

# 创建用户
create user username_ssl@'%' identified by 'password' require ssl;
```

### 刷新权限表

```SQL
flush privileges;
```

## 二、数据库与表显示、创建、删除

### 1. 数据库查看&创建&删除

```SQL
-- 查看数据库
show databases;  

-- 创建库
create database [IF NOT EXISTS] <库名> [character set='utf8'];

-- 删除库
drop database <库名>;  
```

### 2. 表查看、创建、删除

```SQL
-- 显示数据表
use <库名>;
show tables;

-- 创建表：
create table 表名 (字段设定列表) [engine=InnoDB] [charset=utf8mb4];

-- 查看创建表的 DDL 语句
show create table <表名>;

-- 显示表结构
desc <表名>;

-- 删除表
drop table [IF EXISTS] <表名>;  

-- 临时表
CREATE TEMPORARY TABLE <表名>(<字段定义>);
```

**e.g.**

```SQL
CREATE TABLE
    USER
    (
        id INT NOT NULL AUTO_INCREMENT,
        stu_id INT NOT NULL,
        name VARCHAR(30) NOT NULL,
        phone VARCHAR(20),
        address VARCHAR(30) NOT NULL,
        age INT NOT NULL,
        PRIMARY KEY (id),
        UNIQUE KEY `un_stu_id` (stu_id),
        KEY `idx_name` (`name`) USING BTREE
    )
    ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

注意: 为 text 等建立索引时要指定长度 e.g.  `KEY` idx_text`(f_text(64))`

## 三、表复制、备份还原及清除

### 1. 复制表结构

**含有主键等信息的完整表结构:**

```SQL
CREATE table 新表名 LIKE book;
```

**只有表结构，没有主键等信息:**

```SQL
create table 新表名 select * from books;
或
create table 新表名 as(select * from book);
或
create table 新表名 select * from books where1=2;
```

注: `create table <t> select ...` 会造成索引丢失

### 2. 将旧表中的数据灌入新表

```SQL
INSERT INTO <新表> SELECT * FROM <旧表>;
```

### 3. 显示创建表的DDL语句

```SQL
show create table <表名>;
```

### 4. 清空表数据

```SQL
truncate table <表名>;
```

### 5. 备份数据库

备份单个库

```SQL
shell> mysqldump --single-transaction --master-data=2 --default-character-set=utf8 -u root -p <database_name> >database_name.sql
```

备份一个表

```SQL
shell> mysqldump --single-transaction --master-data=2 --default-character-set=utf8 -u root -p <database_name> <table_name> > table_name.sql
```

备份多个库

```SQL
shell> mysqldump --single-transaction --master-data=2 --default-character-set=utf8 -u username -p --databases <dbname1> <dbname2> > Backup.sql
```

备份所有库

```SQL
shell> mysqldump --single-transaction --master-data=2 --default-character-set=utf8 -A -u root -p > back.sql
```

忽略某些库表

```SQL
--ignore-table=performance_schema.* --ignore-table=information_schema.* --ignore-table=sys.* --ignore-table=test.*
```

带上压缩

```SQL
shell> mysqldump -A | gzip >> backup.sql.gz
```

### 6. 还原数据库

```SQL
shell> mysql -u root -p -f [database_name] < backup.sql
```

### 7. 从备份文件抽取数据

提取某个库的所有数据

```SQL
shell> sed -n '/^-- Current Database: `test_restore`/,/^-- Current Database:/p' mysql_back.sql
```

只提取建表语句

```SQL
shell> sed -e'/./{H;$!d;}' -e 'x;/CREATE TABLE `test1`/!d;q' mysql_back.sql
```

只提取数据

```SQL
shell> grep -i 'INSERT INTO `test1`' mysql_back.sql
```

提取所有建库表语句

```SQL
shell> grep -iv 'INSERT INTO `' mysql_back.sql
```

### 8. 导出数据

```SQL
<select语句> into outfile "dest_file";
```

### 9. 导入数据

```SQL
load data infile "<file_name>" into table <table_name>;
```

## 四、修改表的列与表名

### 1. 给列更名

```SQL
alter table <表名称> change <旧字段名称> <新字段名称>
```

### 2. 给表更名

```SQL
alter table <旧表名称> rename <新表名称>
```

### 3. 修改某个表的字段类型及指定为空或非空

```SQL
alter table <表名称> change <字段名称> <字段名称> <字段类型> [not null];
alter table <表名称> modify <字段名称> <字段类型> [not null];
```

### 4. 增加一个字段(一列)

```SQL
alter table 表名称 add column 字段名称 字段类型;
```

加在某个字段后面

```SQL
alter table 表名称 add column 字段名称 字段类型 after 某个字段;
```

加在最前

```SQL
alter table 表名称 add column 字段名称 字段类型 first;
```

### 5. 更改一个字段名字(也可以改变类型和默认值)

```SQL
alter table <表名称> change <原字段名称> <新字段名称 字段类型>;
```

### 6. 改变一个字段的默认值, 该方法不会锁表

```SQL
alter table 表名称 alter 字段名称 set default 值;
```

### 7. 改变一个字段的数据类型

```SQL
alter table <表名称> change column <字段名称> <字段名称> <字段类型>;
```

### 8. 删除字段

```SQL
alter table <表名称> drop column <列名>;
```

## 五 查询表

```SQL
SELECT [DISTINCT] <字段名称,用逗号隔开/*>

FROM <left_table> [<join_type> JOIN <right_table> ON <连接条件>]

WHERE <where条件>

GROUP BY <分组字段>

HAVING <筛选条件>

ORDER BY <排序条件> [desc/asc]

LIMIT n[, m]
```

### 1. GROUP BY 与聚合函数 使用注意点

1 在不使用聚合函数的时候，group by 子句中必须包含所有的列，否则会报错

正确： `select name,age from test group by name,age; //和 select 一样`

2 在 group by 子句中不要加上聚合函数处的列名

### 2. having

SQL 标准
要求 having 必须引用 group 子句中的列或者用聚合函数处理过
后的列。

mysql 对这一标准进行了一些扩展，它允许 having 引
用 select 中检索的列和外部查询中的列。

having 中用到的条件要
么在 group by 中出现，要么在 select 的列中出现，要么在外查
询中出现。

### 3. from

from 子查询时要给数据表指定一个别名。from (select ..) [as] 别名 where...

### 4. union

```SQL
select 语句 union [all] select 语句
```

union 会去重

### 5. join 外连接查询

```SQL
select * from tableA A [left、right] join tableB B on A.id = B.id
```

### 6. join 交叉连接

```SQL
select * from tableA,tableB
select * from tableA cross join tableB
```

逗号与 `cross join` 区别是逗号不能使用 on

结果会有 n * n 条记录（笛卡尔乘积）

### 7. join 内连接

```SQL
select * from tableA A inner join tableB B on A.id = B.id
select * from tableA A inner join tableB B using(id)
```

`using(字段)` 可以合并相同字段,并且符合 A.id = B.id

内连接在没有条件时和交叉连接没有区别。

`STRAIGHT_JOIN` 可以手动指定驱动表

## 六 索引的创建、删除和查看

### 1. 创建索引

***方法一***

```SQL
-- 普通索引
ALTER TABLE 表名称 ADD INDEX index_name (column_list)
-- 唯一索引
ALTER TABLE 表名称 ADD UNIQUE (column_list)
-- 主键索引
ALTER TABLE 表名称 ADD PRIMARY KEY (column_list)
```

***方法二***

```SQL
CREATE INDEX index_name ON 表名称 (column_list)
CREATE UNIQUE INDEX index_name ON 表名称 (column_list)
```

column_list 指出对哪些列进行索引，多列时各列之间用逗号分隔。
索引名index_name可选，缺省时，MySQL将根据第一个索引列赋一个名称。
另外，ALTER TABLE允许在单个语句中更改多个表，因此可以在同时创建多个索引。

### 2. 删除索引

```SQL
-- 删除索引
DROP INDEX index_name ON 表名称;
ALTER TABLE 表名称 DROP INDEX index_name;
-- 删除主键
ALTER TABLE 表名称 DROP PRIMARY KEY;
```

### 3. 查看索引

```SQL
show index from 表名称;
show keys from 表名称;
```

### 4. 手动选择索引

- `USE INDEX` : 向优化器提示如何选择索引
- `IGNORE INDEX` : 忽略索引
- `FORCE INDEX` : 强制使用索引

```SQL
select * from tableA USE INDEX (key1, key2) where key1=1 and key2=2
```

## 七 外键

### 1. 增加外键

建表时

```SQL
constraint 外键名 foreign key(外键字段) references 关联表名(关联字段);
```

修改表

```SQL
alter table 表名 add constraint 外键名 foreign key(外键字段名) references 外表表名(对应的表的主键字段名);
```

## 2. 删除外键

```SQL
ALTER TABLE table-name DROP FOREIGN KEY key-id;
```

## 八 流程控制&函数

### 1 内置函数&方法

#### 1.1 if

```SQL
IF(expr1,expr2,expr3)
```

如果 expr1 是TRUE (expr1 <> 0 and expr1 <> NULL)，则 IF()的返回值为expr2; 否则返回值则为 expr3。

### 1.2 CASE when

```SQL
SELECT CASE 1 WHEN 1 THEN 'one' WHEN 2 THEN 'two' ELSE 'more' END as testCol
```

### 1.3 IFNULL

```SQL
IFNULL(expr1,expr2)
```

假如expr1 不为 NULL，则 IFNULL() 的返回值为 expr1; 否则其返回值为 expr2。

### 2 自定义存储过程&函数

#### 2.1 查看

查询数据库中的存储过程和函数

```SQL
- 存储过程
show procedure status;
select `name` from mysql.proc where db = '<dbname>' and `type` = 'PROCEDURE';

-- 函数
show function status;
select `name` from mysql.proc where db = '<dbname>' and `type` = 'FUNCTION'
```

查看存储过程或函数的创建代码

```SQL
show create procedure <proc_name>;
show create function <func_name>;
```

## 九 视图

### 1. 创建

```SQL
create or replace view <视图名>(<列名 1>,<列名 2>...) as <select 语句>;
```

### 2. 删除

```SQL
drop view <视图 1> [，视图 2....视图 n];
```

## 十 触发器

```SQL
trigger_time: { BEFORE | AFTER } -- 事件之前还是之后触发
trigger_event: { INSERT | UPDATE | DELETE } -- 三个类型
trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

假设定义一个触发器，每次插入把 `ctime` 设为当前时间

```SQL
delimiter // -- 更改结束符
create trigger <触发器名字>
before insert on <表名>
for each row
begin
set new.ctime=now();
end;//
delimiter ;
```

假设定义一个触发器，每次更新把 `mtime` 设为当前时间

```SQL
delimiter // -- 更改结束符
create trigger <触发器名字>
before update on <表名>
for each row
begin
set new.mtime=now();
end;//
delimiter ;
```

## 十一 查看状态

```SQL
# 查看状态
status;
show status;

# innodb 状态
show innodb status;

# 查看参数
show variables like '%参数名称%';

# 查看隔离级别
select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

## 十二 导出导入

### 1. 导出到文件

```SQL
select * into outfile 文件地址 [控制格式] form tableA;
```

***导出到 csv，并压缩***

```SQL
shell> mysql -B -u账号 -p -e "SELECT语句" | sed "s/'/\'/;s/\t/\",\"/g;s/^/\"/;s/$/\"/;s/\n//g" | gzip > data.csv.gz
```

控制格式同导入文件

### 2. 导入文件

```SQL
load data infile 文件名 [replace|ignore] into table 表名 [控制格式]
```

- `replace` 和 `ignore`: 表示对主键重复的数据处理方式
- 控制格式 `fields terminated by '\t' enclosed by '' escaped by '\\'`

e.g.

```SQL
SELECT * INTO OUTFILE '/tmp/data.txt'  
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM tableA;
```

## 十三 统计语句

### 1. 计算最大可能使用内存

***MySQL >= 8***

```SQL
select
(@@key_buffer_size + @@query_cache_size + @@tmp_table_size
+ @@innodb_buffer_pool_size +
@@innodb_additional_mem_pool_size
+ @@innodb_log_buffer_size
+ @@max_connections * (
@@read_buffer_size + @@read_rnd_buffer_size
+ @@sort_buffer_size+ @@join_buffer_size
+ @@binlog_cache_size + @@thread_stack
)
)/1024/1024/1024 as max_mem_G;
```

***MySQL < 8***

```SQL
select
(@@key_buffer_size + @@query_cache_size + @@tmp_table_size
+ @@innodb_buffer_pool_size
+ @@innodb_log_buffer_size
+ @@max_connections * (
@@read_buffer_size + @@read_rnd_buffer_size
+ @@sort_buffer_size+ @@join_buffer_size
+ @@binlog_cache_size + @@thread_stack
)
)/1024/1024/1024 as max_mem_G;
```

### 2. 数据大小

***数据总大小***

```SQL
SELECT ROUND(SUM(DATA_LENGTH)/1024/1024/1024,2) as data_size_G,ROUND(SUM(INDEX_LENGTH)/1024/1024/1024,2) as index_G, ROUND(SUM(DATA_LENGTH+INDEX_LENGTH)/1024/1024/1024,2) as total_size_G,SUM(TABLE_ROWS) as rows FROM information_schema.TABLES;
```

***某个库大小***

```SQL
SELECT ROUND(SUM(DATA_LENGTH)/1024/1024/1024,2) as data_size_G,ROUND(SUM(INDEX_LENGTH)/1024/1024/1024,2) as index_G, ROUND(SUM(DATA_LENGTH+INDEX_LENGTH)/1024/1024/1024,2) as total_size_G,SUM(TABLE_ROWS) as rows FROM information_schema.TABLES WHERE TABLE_SCHEMA='库名';
```

***统计所有库按大小排序***

```SQL
SELECT TABLE_SCHEMA, ROUND(SUM(DATA_LENGTH)/1024/1024/1024,2) as data_size_G,ROUND(SUM(INDEX_LENGTH)/1024/1024/1024,2) as index_G, ROUND(SUM(DATA_LENGTH+INDEX_LENGTH)/1024/1024/1024,2) as total_size_G,SUM(TABLE_ROWS) as rows FROM information_schema.TABLES group by TABLE_SCHEMA order by data_size_G desc;
```

### 3. 统计连接IP

```SQL
select SUBSTRING_INDEX(host,':',1) as ip , count(*) from information_schema.processlist group by ip;
```

### 4. 查看锁

***mysql5.6***

```SQL
SELECT
    r.trx_id waiting_trx_id,
    r.trx_mysql_thread_id waiting_thread,
    r.trx_query waiting_query,
    b.trx_id blocking_trx_id,
    b.trx_mysql_thread_id blocking_thread,
    b.trx_query blocking_query
FROM
    information_schema.innodb_lock_waits w
INNER JOIN
    information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
INNER JOIN
    information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;



waiting_trx_id -- 请求的事物ID
waiting_thread -- 请求的线程ID
waiting_query -- 等待的SQL语句
blocking_trx_id -- 阻塞上面请求的事物的ID
blocking_thread -- 阻塞的线程ID
blocking_query -- 阻塞的当前的SQL，这个是无法看到的，除非SQL还没有执行完成（不一定是该事物当中的最后一条SQL语句）
```

***mysql5.7***

```SQL
select * from sys.innodb_lock_waits;
```

### 5. 查看没有主键的表

```SQL
SELECT
    table_schema, table_name
FROM
    information_schema.TABLES
    WHERE
    table_name NOT IN (
        SELECT DISTINCT
            TABLE_NAME
        FROM
            information_schema.COLUMNS
        WHERE
            COLUMN_KEY = 'PRI')
        AND table_schema NOT IN ('mysql' , 'information_schema','sys', 'performance_schema');
```

### 6. 索引合理性

```SQL
SELECT
    t.TABLE_SCHEMA,t.TABLE_NAME,INDEX_NAME, CARDINALITY, TABLE_ROWS,
    CARDINALITY/TABLE_ROWS AS SELECTIVITY
FROM
    information_schema.TABLES t,
    (
        SELECT
            table_schema,
            table_name,
            index_name,
            cardinality
        FROM information_schema.STATISTICS
        WHERE (table_schema,table_name,index_name,seq_in_index) IN (
            SELECT
                table_schema,
                table_name,
                index_name,
                MAX(seq_in_index)
            FROM
                information_schema.STATISTICS
            GROUP BY table_schema , table_name , index_name
        )
    ) s
WHERE
    t.table_schema = s.table_schema
        AND t.table_name = s.table_name  
        AND t.table_schema = '数据库名' -- 指定某一个库名
ORDER BY SELECTIVITY;
```

- SELECTIVITY越接近1，越合理
- 因为聚合索引会在 statistics 表中产生多条数据，所以 MAX(seq_in_index) 可以拿到完整索引那条

### 7. 统计processlist各个状态数量

```SQL
shell> mysql -uroot -p<password> -e 'show processlist \G' | grep 'State:' | sort | uniq -c | sort -rn
```

## 十四 运维语句&命令

### 1. 更新统计信息

```SQL
analyze table <table_name>;
```

### 2. 重新整理表

```SQL
optimize table <table_name>;
```

### 3. 检查表(MyISAM)

```SQL
check table <table_name>;
```

### 4. 修复表(MyISAM)

```SQL
repair table <table_name>;
```

### 5. 批量检查或修复表

```SQL
# 检查所有表
shell> mysqlcheck -u root -p<password> -A -c  

# 修复所有表
shell> mysqlcheck -u root -p<password> -A -c  
```

### 6. 统计每秒慢日志

```SQL
shell> awk '/^# Time:/{print $3, $4, c;c=0}/^# User/{c++}' slowquery.log
```

### 7. 查看binlog日志

***基于position***

```SQL
mysqlbinlog --no-defaults -v -v --base64-output=DECODE-ROWS --start-position=<start_pos> --stop-position=<stop_pos> <mysql binlog文件> > result.sql
```

***基于时间点***

```SQL
mysqlbinlog --no-defaults -v -v --base64-output=DECODE-ROWS --start-datetime='<开始时间>' --stop-datetime='<结束时间>' <mysql binlog文件> > result.sql
```

***查看某个pos的日志***

```SQL
mysqlbinlog --no-defaults -v -v --base64-output=DECODE-ROWS <mysql binlog文件> | grep -A '20' <pos>
```

### 8. 打开句柄数

**统计各进程打开句柄数：**

```SQL
lsof -n|awk '{print $2}'|sort|uniq -c|sort -nr | head -n 10

# 某个进程
lsof -n|awk '$2=="<某个进程的PID>" {print $2}'|sort|uniq -c|sort -nr | head -n 10
```

***统计各用户打开句柄数***

```SQL
lsof -n|awk '{print $3}'|sort|uniq -c|sort -nr

# mysql用户
lsof -n|awk '$3 == "mysql" {print $3}'|sort|uniq -c|sort -nr
```

***统计各命令打开句柄数***

```SQL
lsof -n|awk '{print $1}'|sort|uniq -c|sort -nr
```

### 9. 导出用户权限(shell脚本)

```SQL
#/bin/bash

user='username'
pass='password'
sock='socket'

expgrants()  
{  
  mysql -B -u"${user}" -p"${pass}" -S"${sock}" -N $@ -e "SELECT CONCAT(  'SHOW CREATE USER   ''', user, '''@''', host, ''';' ) AS query FROM mysql.user" | \
  mysql -u"${user}" -p"${pass}" -S"${sock}" -f  $@ | \
  sed 's#$#;#g;s/^\(CREATE USER for .*\)/-- \1 /;/--/{x;p;x;}'  
  
  mysql -B -u"${user}" -p"${pass}" -S"${sock}" -N $@ -e "SELECT CONCAT(  'SHOW GRANTS FOR ''', user, '''@''', host, ''';' ) AS query FROM mysql.user" | \
  mysql -u"${user}" -p"${pass}" -S"${sock}" -f  $@ | \
  sed 's/\(GRANT .*\)/\1;/;s/^\(Grants for .*\)/-- \1 /;/--/{x;p;x;}'  
}  
```

然后执行脚本

### 10. 批量杀连接

***方法一***

```SQL
select concat('KILL ',id,';') from information_schema.processlist where user='某个用户' into outfile '/tmp/kill.txt';
source /tmp/kill.txt;
```

***方法二***

```SQL
mysqladmin -uroot -p processlist|awk -F "|" '{if($3 == "要杀的连接的用户")print $2}'|xargs -n 1 mysqladmin -uroot -p kill
```
