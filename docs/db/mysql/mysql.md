#  mysql

## 1.引入

### 数据库分类

#### 关系数据库

- Mysql、Oracle、SQLlite
- 通过表和表的之间、行和列之间的关系来进行数据存储

#### 非关系 not only sql

- Reids、MongoDB
- 对象存储，存储对象自身的属性

#### RDBMS 关系型数据库管理系统

DDL database define language

DML * manage *

DQL * query * 

DCL * control * 

## 2 操作数据库

### 2.1数据库

#### 新建

`create database [if not exists] dbName;`

#### 删除

`Drop database [if exists] dbname;`

### 2.2表 列类型

#### 数值

| Type        | Storage (Bytes) | Minimum Value Signed | Minimum Value Unsigned | Maximum Value Signed | Maximum Value Unsigned |
| :---------- | :-------------- | :------------------- | :--------------------- | :------------------- | :--------------------- |
| `TINYINT`   | 1               | `-128`               | `0`                    | `127`                | `255`                  |
| `SMALLINT`  | 2               | `-32768`             | `0`                    | `32767`              | `65535`                |
| `MEDIUMINT` | 3               | `-8388608`           | `0`                    | `8388607`            | `16777215`             |
| `INT`       | 4               | `-2147483648`        | `0`                    | `2147483647`         | `4294967295`           |
| `BIGINT`    | 8               | `-263`               | `0`                    | `2^63-1`             | `2^64-1`               |

- tinyint 1 字节
- smallint 2 
- mediumint 3
- **int 4**  int(4) 于int(M)只与0填充有关，最大显示长度有关。
- **bigint** 8

- float 4 
- double 8
- decimal 字符串浮点数，做金融运算

#### 字符串

- char 0-255 
- **varchar 0-65535 **常用变量
- tinytext  2*8 -1
- **text 2*16 -1**  保存大文本

#### 时间

- date YYYY-MM-DD 日期
- time  HH-MM-SS 时间
- datetime YYYY-MM-DD HH:MM:SS 
- timestamp 时间戳 1970.1.1 较为常用
- year 

#### null

- 没有值

###  2.3表 字段属性

#### unsigned

- 无符号整形，不可为负数

#### zerofill

- 0填充
- 不足的位数用0来填充
- eg: int(3), 5 , 005

#### autoincrement 

- 用来设置唯一主键 
- 自定义起始值

#### Not Null 

- 不赋值为错

#### default value

### 2.4创建表

```sql
CREATE TABLE IF NOT EXISTS `student`(
    `id`       INT(4)      NOT NULL AUTO_INCREMENT COMMENT 'id',
    `name`     VARCHAR(30) NOT NULL DEFAULT 'unkonwnuser' COMMENT 'name',
    `pwd`      VARCHAR(30) NOT NULL DEFAULT '123456' COMMENT 'password',
    `sex`      VARCHAR(2)  NOT NULL DEFAULT '女' COMMENT 'sexual',
    `birthday` DATETIME             DEFAULT NULL COMMENT 'birthday',
    `address`  VARCHAR(100)         DEFAULT NULL COMMENT 'home_address',
    `email`    VARCHAR(50)          DEFAULT NULL COMMENT 'email',
    PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```

格式

```sql
CREATE TABLE IF NOT EXISTS `student` (
	`字段` 列类型 [属性] [索引] [注释],
	..... 
  `字段` 列类型 [属性] [索引] [注释],
  PRIMARY KEY (`key1`, `key2`)
)[表类型][字符集设置][注释]
```

```sql
show  create  database `school`;

show create table student;

desc student;
```

### 2.5数据库引擎

#### innodb & myisam

|            | MYISAM         | INNODB               |
| ---------- | -------------- | -------------------- |
| 事务支持   | 不支持         | 支持                 |
| 数据行锁定 | 不支持（表锁） | 支持                 |
| 外键       | 不支持         | 支持                 |
| 全文索引   | 支持           | 不支持               |
| 表空间大小 | 较小           | 较大，约为MYISAM 2倍 |

- MYISAM 节约时间，速度快
- INNODB 安全性、事务处理、外键支持多表多用户

> **在物理空间存在的位置**
>
> 数据库本质是文件存储

Mysql引擎在物理文件上的区别

- InnoDB在数据库表中只有一个*.frm文件，以及上级目录下的
  - *.idb 
- myIsam
  - *.frm - 表结构定义文件
  - *.myd - 数据文件
  - *.myi - 索引文件



#### 字符集编码

```sql
charset=utf8
```

### 2.6修改和删除表

```sql	
alter table PAGENAME 操作 ;

-- 重命名表
alter table `teacher` rename as `teacher1`;
alter table `teacher1` rename as `teacher`;

-- 增加新的字段
alter table `teacher` add `city` varchar(20) default null comment 'city';

alter table `teacher` change age1 age int(10); -- 重命名字段
alter table `teacher` modify  age int(2); -- 修改约束
alter table `teacher` drop age; -- 删除字段

-- 删除表
drop table if exists teacher;
```

注意事项：

- `` 字段名
- 注释使用 --
- sql 关键字大小不敏感

## 3.mysql数据库管理

### 3.1 外键 

学生的grade列，引用年纪表的id

```sql
create table `grade` (
    `gradeID` int(10) not null auto_increment comment 'the id of grade',
    `gradeName` varchar(50) not null comment 'name of grade',
    PRIMARY KEY (`gradeID`)
)engine=innoDB default charset=UTF8MB3;

-- 学生表的gradeID字段 要引用gradeid
-- 定义外键key
-- 给这个外键添加约束 reference 引用
CREATE TABLE `student` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'id',
  `name` varchar(30) NOT NULL DEFAULT 'unkonwnuser' COMMENT 'name',
  `pwd` varchar(30) NOT NULL DEFAULT '123456' COMMENT 'password',
  `sex` varchar(2) NOT NULL DEFAULT '女' COMMENT 'sexual',
  `birthday` datetime DEFAULT NULL COMMENT 'birthday',
  `address` varchar(100) DEFAULT NULL COMMENT 'home_address',
  `email` varchar(50) DEFAULT NULL COMMENT 'email',
  `gradeID` int(10) not null  comment 'the grade',
  PRIMARY KEY (`id`),
  key `FK_gradeID` (`gradeID`),
      constraint `FK_gradeID` foreign key (`gradeID`) references `grade`(`gradeID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;
```

删除先删除从表后删除主表。

```sql
-- 创建表的时候没有关系
ALTER Table `student` ADD COLUMN `gradeID` INT(10) NOT NULL COMMENT 'grade id';
ALTER TABLE `student` ADD CONSTRAINT `FK_gradeID` FOREIGN KEY(`gradeID`) REFERENCES `grade`(gradeID);

```

不建议使用，数据库过多会造成困扰。

**最佳实践**

- 数据表只有本身的表
- 如果要使用多张表则使用外键，通过程序逻辑来进行实现

### 3.2DML语言

> 数据操作语言，数据存储，数据管理

- #### insert

- #### update

- #### delete

### 3.3 插入

`INSERT INTO table_name([word1],[word2]) VALUES('value1', 'value2', 'value3')`

- 字段可以省略，但是值需要一一对应
- 可以同时插入多条数据

```sql

-- 插入语句
-- insert into table_name([col1]) values('value1'),('values')
INSERT INTO `grade`(`gradeName`) values ('大四');
-- insert into 不指定字段则会一一匹配
INSERT INTO `grade`(`gradeName`)
value ('大三'),
    ('大二'),
    ('大一');


INSERT INTO `student`(`name`,`pwd`,`sex`)
VALUES ('陈秋实','123asdzxc','男'),('zjj','123asdzxc','男');
```

### 3.4 修改

`UPDATE table_name SET column_name = value WHERE 条件[]`



```sql
-- 修改student名字
UPDATE `student` SET `name`='zhangshuntao' where id =1;
-- 不指定条件的情况下，会修改掉所有的表

-- 修改多个字段
UPDATE `student` SET `name` = 'qiusiu', `email`='123@qq.com' where  id = 1;

```

**条件：**Where 子句

| **操作符**      | 含义     | 范围            | 结果   |
| --------------- | -------- | --------------- | ------ |
| =               | 等于     | 5=6             | false  |
| !=              | 不等于   | 5!=6            |        |
| >               |          |                 |        |
| <               |          |                 |        |
| <=              |          |                 |        |
| >=              |          |                 |        |
| BETWEEN ... and | 闭合区间 | BETWEEN 2 and 5 | [2, 5] |
| AND             | 和       |                 |        |
| OR              | 或者     |                 |        |

```sql
-- 通过多个条件定义
UPDATE `student` SET `name` = '多条语句修改的' where `name`='zjj' AND `pwd`='9999';
```

`UPDATE table_name SET column_name = value WHERE 条件[]`

注意

- column_name 带上 ``
- condition，帅选的条件，如果没有指定则会修改所有的列
- value可以是一个具体的值，也可以是一个变
- 多个设置的属性之间，用英文逗号隔开

### 3.5 删除

`DELETE FROM table_name WHREE [condition]`

```sql
delete from `student;
delete from `student` where id = 1;
```

> **TRUNCATE** 完全情况数据库，结构和索引不会发生变化

`TRUNCATE table_name` 

- 相同点：都能删除数据，但是不会删除表结构
- 不同点：
  - TRAUNCATE 会重置自增计数器
  - TRUNCATE 不影响事务

了解，`DELETE 删除问题`，重启数据库，现象：

- InnoDB 自增列会从1开始，因为InnoDB的自增计数器存在于内存中
- MyISAM 继续从上一个自增量开始，因为自增计数存在于文件之中

## 4.DQL数据库查询语言

### 4.1 DQL

数据查询语言 date query language

- 所有查询操作都用Select
- 简单的查询和复杂查询都能做
- **数据库核心语言，最重要的语言** 
- 使用最高频

语法`SELECT 字段... FROM 表`

```sql
-- 查询全部学生
SELECT * from `student`;
-- 查询制定字段
SELECT `studentno` a, `studentname` b FROM  `student`;
-- 给结果使用别名, 给表其别名
SELECT `studentno` AS '学号', `studentname` AS '名字' From `student`;

-- 函数 Concat(a, b)
SELECT CONCAT('name: ', studentname) AS name FROM `student`;
```

> 如果字段名不够好理解，可以使用AS指定字段名

### 4.2 查询指定字段

去重 SELECT DISTINCT

```sql
SELECT * from result; -- 查询全部成绩
-- 查询有那些同学参加了考试
select `studentno` from result;
select DISTINCT  `studentno` from result;
```

> 数据库中的列

```sql
-- select vserion()

select  version(); -- 函数
select 100*3-1 AS result; -- 表达式
SELECT @@AUTO_INCREMENT_INCREMENT; -- 变量

-- 学院考试成绩+1分查看
SELECT `studentno`,`studentresult` AS beforeResult, `studentresult`+1 AS AfterResult FROM result;
```

数据库中的表达式：文本值，列，Null，函数，计算表达式，系统变量……

`SELECT 表达式 FROM 表`

### 4.3 where条件字句

所有能够使用在检索数据中的值

有一个或者多个表达式组成

| 运算符 | 语法    | 描述 |
| ------ | ------- | ---- |
| and    | a and b |      |
| or     | a or a  |      |
| Not    | not a   | !    |

- 尽量使用单词

```sql
-- where
SELECT studentno, `studentresult` from result
where studentresult>=95 and studentresult <=100;

-- between and 查询区间
select  studentno, studentresult from result
where studentresult between 95 and 100;

-- 除了1000号学生以外的成绩
select studentno, studentresult AS 高分科目 from result
where studentno != 1000 and studentresult between 90 and 100;
select studentno, studentresult from result
where not studentno = 1000;
```

 

模糊查询：比较运算符

| 运算符      | 语法              | 描述                           |
| ----------- | ----------------- | ------------------------------ |
| IS NULL     | a is null         |                                |
| IS NOT NULL |                   |                                |
| BETWEEN     | a between b and c |                                |
| **like**    | a like b          | SQL 如果A能匹配到B，则结果为真 |
| **in**      | a in (a1,a2,a3)   | a在a1,a2,a3中则为真            |

```sql
-- 模糊查询
-- 查询姓刘的同学
-- like 结合 %（0到任意一个字符）_（一个字符）
SELECT `studentno`, studentname from student where studentname like '李%';
-- like 结合 %（0到任意一个字符）_（一个字符）
SELECT `studentno`, studentname from student where studentname like '李___';
-- 查询名字中有牛字的同学
SELECT `studentno`, studentname from student where studentname like '%牛%';

-- in 包含关系
-- 查询1001，1002，1003号同学
select `studentno`, `studentname` from `student` where studentno in (1001, 1002, 1003);
-- 查询在北京的学生
select `studentno`, `studentname`, `address` from `student` where address in ('北京朝阳');

-- null 和not null
-- 查询地址为空的学生
select studentno, studentname, address from student where address = '';

-- 查询出生日期的同学
select studentno, studentname from student where borndate is null;
```

### 4.4 联表查询

> Join

左表 ，右表

LeftJoin InnerJoin Right join

![img](/Users/chenqiushi/Documents/工作台/博客文章/7_joins.png)

```sql
-- 查询参加了考试的同学，学号， 名字，科目编号，分数
/*
1. 需要那些表
2，需要使用哪一种连接查询? 7中
确定交叉点，数据相同的
判断条件 学生表中的studentNO = 成绩表中的studentNo
*/
SELECT s.studentno, s.studentname, subjectno, studentresult
from student s
inner join result r
on s.studentno = r.studentno;

select s.studentno, studentname, subjectno, studentresult
from student s
right join result r
on r.studentno = s.studentno;
-- 
select s.studentno, studentname, subjectno, studentresult
from student s
left join result r
on r.studentno = s.studentno;
```

| **操作**   | **描述**                                               | 基准         |
| ---------- | ------------------------------------------------------ | ------------ |
| Inner join | 表中至少有一个匹配就会返回                             |              |
| left join  | 会从左表中返回所有的值，右表中没有匹配的值也会返回出来 | 已左表为基准 |
| right join | 会从右表中返回所有的值，左表中没有匹配的值也会返回出来 | 以右表为基准 |

```sql
-- 查询缺考的同学
select s.studentno, studentname, subjectno, studentresult
from student s
left join result r
on r.studentno = s.studentno
where studentresult is null;

-- 学号，学生姓名，科目名，分数
select s.studentno, studentname, subjectname, studentresult
from student s
inner join result r
on r.studentno = s.studentno
left join subject j
on r.subjectno = j.subjectno
```

#### 4.4.1自连接

自己的表和自己的表连接，核心：一张表拆分成两张一样的表。

```sql
select c1.categoryname as p, c2.categoryname as s
from category c1, category c2
where c1.categoryid = c2.pid;
```



```sql
SELECT [ALL | DISTINCT]
{* | table.* | [table.filed1[as alias1][,table.filed2[as alias2][,...]]]}
FROM table_name [as table_alias]
		[left | right | inner join table_name2] -- 联合查询
		[where ...]
		[Group by ...]-- 按哪些个计算来分组
		[Having] -- 过滤分组需要的次要条件
		[order by ...] -- 排序条件
		[limit {[offset,]row_count | row_countOFFSET offset}]; -- 指定记录需要的记录从哪一条道那一条
```

### 4.5 分页和排序

```sql
-- 分页 和 排序
-- 升序ASC， 降序DESC
-- 网页：当前、总的页数、页面大小
-- LIMIT 0,5 从 1 ～5
-- LIMIT 1，6 从 2～6
select r.studentno, studentname, studentresult
from result r
inner join student s on r.studentno = s.studentno
order by studentresult
limit 1 offset 4;

-- 第一页 0-5
-- 第二页 5-5
-- 第三页 10-5
-- 第n页 (n-1)*5 - 5 (pageNum-1)*pageSize,pageSize
-- totalPage = total / pageSize 
```

`limit 起始值 页面大小`

### 4.6 子查询

where ( )

在where语句中在再嵌套一个查询

```sql
-- 高等数学-1的科目
select distinct s.studentno, studentname, subjectno
from student s
         inner join result r on s.studentno = r.studentno
where studentresult >= 80
  and subjectno = (
    select subjectno
    from subject
    where subjectname = '高等数学-1'
);


select distinct studentno, studentname
from student
where studentno in (
    select studentno
    from result
    where studentresult >= 80
      and subjectno = (
        select subjectno
        from subject
        where subjectname = '高等数学-1'
    )
);



-- 查询课程为高等数学-2 且分数不小于80分的同学学号和姓名
select distinct r.studentno, studentname
from student s
         inner join result r
                    on r.studentno = s.studentno
         inner join subject sub
                    on r.subjectno = sub.subjectno
where subjectname = '高等数学-1'
  and studentresult > 80;
```

## 5. mysql 函数

### 5.1 常用函数

```sql
-- 常用函数
select abs(-8)
select CEILING(9.4);-- 向上区镇
select FLOOR(9.1);
select RAND();
-- 判断数的符号
select sign(-9);
-- 字符串函数
select char_length('即使我爱你');
select concat('1', '-', ' ', '8');
select insert('qiuqiu', 1, 3, '123');
select lower('aBBBc');
select upper('abababab');
select instr('hello', 'lo');
select replace('woaiwoainiainiaini', 'ai', '爱');
select substr('woaiwoaini', 3);
select reverse('123456');
-- 查询姓李的同学
select REPLACE(studentname,'李','li')
from student
where studentname like '李%';

-- 时间和日期函数
select current_date();
select current_time();
select now();
select localtime();
select sysdate();
select year(now());
select day(now());

-- 用户
select version();
select user();
select system_user();
```

### 5.2 聚合函数

| 函数名称    | 描述 |
| ----------- | ---- |
| **COUNT()** |      |
| SUM()       |      |
| AVG()       |      |
| MAX()       | `    |
| MIN()       |      |
| ...         |      |

```sql

-- 聚合函数
-- 都能够做统计，
select count(gradeid)
from student; -- 指定列，会忽略所有的null值，当列字段为主键的时候，则是最快的。
select count(*)
from student; -- 不会忽略null值, 本质都是计算行数，count*会计算所有的行数
select count(1)
from student; -- 不会忽略null值，本质都是计算行数，count(1)会把所有的归位一列只会走一次

select sum(studentresult) as sum, avg(studentresult) as avg, max(studentresult) as max, min(studentresult) as min
from result;

-- 查询不同课程的平均分，最高分，最低分
-- 不同的课程进行分组
select subjectname as name, avg(studentresult) as avg, max(studentresult) as max, min(studentresult) as min,count(1) as count
from result r
inner join subject s on r.subjectno = s.subjectno
group by r.subjectno -- 通过什么字段进行分组
having avg > 80; -- 分组条件
```

### 5.3 数据库级别的md5加密

什么是MD5,信息摘要算法。

MD5不可逆，具体的值的md5是一致的

```sql
show create table testmd5;

insert into testmd5
values (2, 'zhangsan', '123'),
       (3, 'zhangsa1n', '123');

update testmd5 set pwd=md5(pwd) where  id = 1;
update testmd5 set pwd=md5(pwd) where not id = 1;

-- 插入的时候进行加密

insert into testmd5(name, pwd)
values ('xiaoming',md5('123456'));

-- 如何校验，将传入进来的密码进行加密，对比加密之后的值
select * from testmd5 where  name = 'zhangsan' and pwd=md5('123');
```

## 6.事务

### 6.1 什么是事务（transaction）

多条sql，要么成功，要么都失败

将一组sql，放在一个批次中去执行

> 事务原则：ACID
>
> **原子性（Atomicity），一致性（Consistency），隔离性（Isolation），持久性（Durability）**

**原子性**： 要么都完成，要么都失败

**一致性**：事务前后数据的完整性要保持一致

**持久性**：事务的结束状态不会随着外界状态导致数据丢失

- 没有提交，恢复到原来状态
- 一旦提交，就会持久化

**隔离性**：多个用户用户并发操作保证事务相互隔离，

### 6.2 隔离导致的问题

#### 脏读

一个事务读取到了另外一个事务没有提交的数据。

#### 不可重复度

一个事务在多次读取表中一行数据，多次读取结果不同。（不一定不对，不是错误）

#### 幻读

在一个事务里读取到了别的事务插入的数据，导致前后读取不一致。

### 6.3 执行事务

> 执行事务

```sql
-- mysql是默认打开事务的
-- 手动处理事务
set autocommit = 0; -- 关闭
-- 事务开启
start transaction;

-- 提交
commit;

-- 回滚
rollback ;
set autocommit  = 1; -- 开启
--
savepoint; -- 设置一个事务的保存点
rollback to savepoint ; -- 回滚保存点
release savepoint ;-- 删除保存点
```

```sql
--
drop database shop;
create database shop char set utf8 collate utf8_general_ci;
use shop;

CREATE TABLE `account`
(
    `id`    int           NOT NULL AUTO_INCREMENT,
    `name`  varchar(30)   NOT NULL,
    `money` decimal(9, 2) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb3

show create table account

insert into account(name, money)
values ('qiuqiu', 90000);

update account set money = 2000.00 where id = 1;

insert into account(name, money)
values ('zhuzhu', 10000.00);

-- 模拟转账
set autocommit = 0;
start transaction;
update account set money=money-500 where name='qiuqiu';
update account set money=money+500 where name='zhuzhu';
COMMIT; -- 事务一旦提交，就会被被持久化
rollback;

set autocommit =1;
```

## 7.索引

### 7.1 索引的分类 

> **索引是帮助mysql高效获取数据的数据结构。**
>
> 
>
> codinglabs mysql索引背后的数据结构计及算法原理

- 主键索引 primary key
  - 唯一标识，主键不可重复
- 唯一索引 unique key
  - 避免重复的列，唯一可以重复，多个列都可以都可可以为唯一索引。
- 常规索引 key
  - 默认的，index或者key关键字来设置
- 全文索引 fulltext
  - 快速定位索引

```sql

-- 索引使用
-- 1。在创建时增加
-- 2。在创建完成后通过alter操作表加入

use school;
show indexes from student;
-- 增加一个缩影
alter table school.student add fulltext index studentname1 (studentname);
-- 删除一个缩影
alter table school.student drop key studentname1;
-- explain 分析sql执行状态
explain select *from student;

explain select *from student where match(studentname) against('张伟');
```

### 7.2测试索引

```sql
-- 创建用户表
CREATE TABLE `app_user`
(
    `id`          bigint unsigned NOT NULL AUTO_INCREMENT,
    `name`        varchar(50)          DEFAULT '' COMMENT 'name',
    `email`       varchar(50)     NOT NULL COMMENT 'mail',
    `phone`       varchar(20)          DEFAULT '',
    `gender`      tinyint unsigned     DEFAULT '0',
    `password`    varchar(100)    NOT NULL,
    `age`         tinyint              DEFAULT '0',
    `create_time` datetime             DEFAULT CURRENT_TIMESTAMP,
    `update_time` timestamp       NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci;
```

插入数据

```sql
-- 插入一百万条数据
delimiter $$ -- 写入函数
create function mock_data() returns int
begin
    declare num int default 1000000;
    declare i int default 0;
    while i < num
        do
            -- 插入语句
            insert into app_user(name, email, phone, gender, password, age)
            values (concat('user', i), '123@qq.com',
                    concat('18', floor(rand() * ((999999999 - 100000000) + 100000000))), floor(rand() * 2), uuid(),
                    rand() * 100);
            set i = i + 1;
        end while;
    return i;
end;

```

```sql
select mock_data();

explain select * from app_user where name = 'user9999'; -- 993261rows 1 s 156 ms
explain select * from app_user where name = 'user999999'; -- 993261  1 s 156 ms

select * from student; -- 24 ms

drop index id_app_user_name on app_user;

create index id_app_user_name on app_user(name);

alter table app_user add index id_app_user_name (name);

explain select * from app_user where name = 'user9999'; -- 27ms 1rows

explain select age as age
from app_user
order by age desc
limit 1 offset 2;
```



建立索引能够极大的缩短查询时间

- 需要建立在很少修改的字段上。

### 7.3索引原则

- 不要对经常变动的数据建索引
- 索引不是越多越好
- 索引一般建立在经常 查询的数据上。

> 索引的数据结构
>
> - btree： Innodb
> - hash

认真读：https://blog.codinglabs.org/articles/theory-of-mysql-index.html

## 8.权限管理

### 8.1用户管理

用户表：mysql.user

对这张表进行增删改查，

```sql
create user cc_test identified by '123456';

set password = PASSWORD('123456');

set password for cc_test = PASSWORD('123456');

rename user cc_test to cc_test2;

-- 
grant all privileges on *.* to cc_test2;

show grants for cc_test2

show grants for root;

revoke all privileges  on *.* form cc_test2;

drop user cc_test;
```

### 8.2数据库备份

为什么要备份，：

- 防止数据丢失
- 数据专业

数据库备份方式：

- 拷贝物理文件
- 可视化工具进行备份和导出
- 命令行 

```bash
> mysqldump -h hostname -u root -p password database_name table_name > **.sql

# 倒入备份文件
soucre path_sql.sql

# mysql
```



## 9.数据库设计

### 9.1设计数据库

**糟糕的数据库设计：**

- 数据段重复，冗余
- 数据库插入和删除产生异常。【屏蔽使用外键】
- 程序性能差



**良好的数据库设计**

- 存储中空间
- 保证数据库的完整性



**软件开发中关于数据库的设计**：

- 分析需求： 分析业务和需求处理的数据库的绣球
- 概要时间：设计关系图E-R图



**设计一个数据库**：一个博客

- 表
  - 用户表
  - 分类表（文章分类，创建者）
  - 文章表（文章信息）
  - 友链表
- 标识实体类

### 9.2 三大范式

> 三大范式

1. #### 第一范式

   **数据表的每一列都是不可分割的原子数据项**

2. #### 第二范式

   前提：满足第一范式，

   属性完全依赖于主键，**每一张数据表只描述一件事情**

3. #### 第三范式

   前提：满足第一范式，和第二范式

   **消除依赖和传递性**，属性之间不依赖于其他非主属性



#### 平衡规范于性能之间的问题

关联查询的表不可以超过三张

- 考虑到体验、目标以及需求，性能更加重要
- 冗余字段能够减少查询，提高性能
- 在保证性能的前提下，适当考虑规范性
- 增加计算列，大数据量减少到小数据量。



## 10其他

### sql注入

web应用没有对用户输入进行过滤，在事先后端程序中的sql中插入sql语句来执行非法的操作。

```sql
-- before
select * from users where name = a and pwd = 1123;
-- after
select * from users where name = a or 1 = 1 and pwd = 123 or 1=1;
```

使用预编译、以及对特殊字符进行过滤等方式来进行应对。