---
title: MySQL学习总结
date: 2018-10-16 20:13:18
tags:
---

总结了常用的SQL语法和MySQL的实际应用

### 1.SQL介绍

数据库~database-数据的仓库(集散地)
通过数据库可以实现数据的持久化
1970s IBM - 关系型数据库
理论基础:关系代数和集合论
具体表象:用二维表来组织数据(行，列)
行:记录 - 实体
列:字段 - 实体的属性
关系型数据库编程语言 - SQL(结构化查询语言 structure query language)
 -DDL(数据定义语言):create / drop /alter
 -DML(数据操作语言):insert / delete /update /select
 -DCL(数据控制语言):grant(授权) / revoke(召回权限)

关系型数据产品:
Oracle - Orale - Oracle 12c
MySQL - Oracle 
SQLServer
PostgreSQL
DB2

非关系型数据库:
MongoDB:文档数据库，适合数据体量大价值低的数据
Redis:适合高速缓存服务(内存中的键值对数据库)
ElasticSearch:搜索引擎

DB - Database - 一堆文件
DBMS -Database Management System - 软件
DBA - Database Administrator - 人
DBS - Database System

查看帮助，在MySQL自带的命令行界面输入help <命令>;或者? <命令>;查看命令的用法

图形化MySQL客户端工具
```sql
-Navicat for MySQL
-Toad for MySQL
-SQLyog
```

E-R:实体关系图
​	一对一:不建立任何键
​	一对多:在多的一边建立外键，关联一的主键
​	多对多:新建一个表,这个表的两个外键分别对应多对多的两个主键。

生活中的
​	一对一:人和身份证，丈夫和妻子
​	一对多:员工和公司
​	多对多:老师和学生

外键 - 外来的主键 - 参照完整性
​	
​数据的完整性
	- 实体完整性:每一条记录都是独一无二的没有冗余
		主键/唯一索引(唯一约束)
	- 参照完整性:B表参照了A表，A表没有的记录，在B表中绝不能出现
		外键
	- 域完整性:录入的数据都是有效的
		数据类型/非空约束/默认值约束/检查约束(MySQL中无效)
数据的一致性

设置外键约束:
alter table <表名> add constraint <约束名> foreign key(<字段名>) references <表名>(字段名);
设置唯一性约束:
alter table <表名> add constraint <约束名> unique(<字段名>);
删除外键约束:
alter table <表名> drop foreign key(<字段名>);
删除唯一性约束:
alter table <表名>drop index <约束名>;
修改字段名:
alter table <表名> change 字段名 新字段名 约束条件;
```mysql
alter table tb_student change teaname tname varchar(10) not null;
```
在代码中添加注释--comment:这样的注释在开发过程中能看到
```mysql
create table tb_student
(
stuid int not null comment '学号',
stuname varchar(4) not null comment '学生姓名',
gender bit default 1,
birth date,
addr varchar(50),
primary key(stuid);
-- comment是在代码中添加注释
```




### 2.DDL(数据定义语言)

#### 2.1 创建数据和删除数据库

cretae databse <数据库名>

drop database <数据库名> [if exists] <数据库名>   在删除数据库前判断数据库是否存在(不要在实际生产中使用)

```mysql
create database tb_student ;

drop databse tb_student if exists tb_student;
```

create table <表名> (),

```mysql
create table tb_student
(
stuid int not null comment '学号',
stuname varchar(4) not null comment '学生姓名',
gender bit default 1,
birth date,
addr varchar(50),
primary key(stuid);

-- comment是在代码中添加注释
```

连接查询:
```mysql
select tname,couname from tb_teacher,tb_cource,tb_score
where tb_teacher.teaid=tb_cource.tid and tb_score.cid=tb_course.couid;
```

#### 2.2 修改表

向原表中添加新列

修改表(添加列)alter table <表名> add column <列名> <数据类型> [约束条件]

```mysql
alter table tb_student add column tel char(11) not null;
```

修改表(删除列)alter table <表名> drop column<列名>

```mysql
alter table tb_student drop column brith;
```

### 3.DML(数据操作语言)

判断一个字段是否为Null,不能用等号和不等号，要用is(是)或者is not(不是)

#### 3.1 insert(插入数据)

插入数据:insert [into] <表名> values((行记录1),(行记录2),...);

```mysql
insert into tb_student values(1001,'张三',1,'12922221234','四川');
```

插入数据 insert [into] <表名>(列名1,列名2,...) values((行记录1),(行记录2),...);

```mysql
insert into tb_student (stuname,stuid,gender) values('李四',1002,1);
```

#### 3.2 delete(删除数据,谨慎使用)

delete from <表名> <约束条件>

删除数据必须要有约束条件，要不然就会把整个表清空
如果真的要删除表数据建议使用一下的语句:
truncate table <表名> 这个叫做截断数据，就是删除全表

```mysql
delete from tb_student where stuid=1001;
```

#### 3.3 update(更新数据)

update <表名> set 列名=新值 ,... [约束条件]

   ```mysql
update tb_student set addr='贵州' where stuid=1002;
   ```

#### 3.4 搜索数据(查询数据)

查询表中所有的数据:select * from <表名> 

```mysql
select * from tb_student;
```

查询表中的部分数据(投影):select 列名 [as] 别名,.... from <表名>

```mysql
select stuname,addr,tel from tb_student where stuid=1002;

select stuname as 姓名, gender as 性别 from tb_student;
-- as可以起别名,也可以不用加直接跟别名在列名后面

select stuname 姓名, if(gender, '男', '女') 性别 from tb_student; --这种只适合mysql语法
select stuname 姓名, case gender when 1 then '男' else '女' end as 性别 from tb_student; 
-- 标准sql 分支结构实现输出男和女
-- select 列名1,case 列名2 when 条件 then "满足条件的输出" else "不满足条件的输出" from <表名>#
```

对列做运算 select concat(列1,列2,...) [[as][别名]] from <表名>

```mysql
select concat(stuname,tel) as 特点 from tb_student; 
```

筛选 select * from <表名> [筛选条件]

```mysql
select * from tb_student where stuid>1001
```

模糊查询 select * from <表名> where <列名>like '值%';

```mysql
select * from tb_student where stuname like '%白%';
-- 模糊查询 
-- %是一个通配符表示0个或者任意多个字符
-- _是一个通配符表示任意一个字符 
```

排序 select * from <表名> order by 列名 [desc(降序)|asc(升序)]

```mysql
select * from tb_student order by stuid desc;
```

限定查询(分页查询)

```mysql
select * from <表名> limit <n>；
n表示每次查询的记录数
select * from <表名> limit <start,n>;
start表示从从第start+1条记录开始查询n条记录

select * from <表名> limit n offset start;
start:跳过的记录数
n:要查询的记录数

select * from tb_student limit 3,3;
```

聚合函数
max()找最大
min()找最小
avg()求平均
sum()求和
count()计数
### 4.DCL(数据控制语言)
grant/revoke/start transaction/commit
#### 4.1 grant
```mysql
-- 创建名为hellokitty的用户
create user 'hellokitty'@'ip地址'identified by '123123';
-- ip地址可以是特定的ip也可以用%表示任意ip
-- indentified by后面是指定的密码

-- 授权
grant select on srs.* to 'hellokitty'@'%';
-- 把srs下的所有的查询权限授权给hellokitty对象 ，srs.*表示所有表

grant insert,delete,update on srs.* to 'hellokitty'@'%';
-- 把添加数据，删除数据，更新数据的的srs下的所有表的权限授权给hellokitty

grant drop,create,alter on srs.* to 'hellokitty'@'%';
-- 将srs数据的drop,create,alter权限授权给hellokitty

grant all privileges on srs.* to 'hellokitty'@'%';
-- 将srs数据库的所有权限授权给hellokitty,但是hellokitty不能授权给其他用户

grant all privileges on srs.* to 'hellokitty'@'%' with grant option;
-- 将srs数据库的所有权限授权给hellokitty，并且hellokitty也可以授权给其他用户

```

#### 4.2 revoke

```mysql
revoke all privileges on srs.* from 'hellokitty'@'%';
-- 将hellokitty的所有权限全部召回
```

#### 4.3 事务控制

```mysql
begin; -- 开启一个事务环境也可以写start transaction
update tb_score set mark=mark-2 where sid=1001 and mark is not null;
update tb_score set mark=mark+2 where sid=1002 and mark is not null;
commit; -- 提交
rollback; -- 事务回滚
```

#### 4.4 MySQL隔离级别

事务(transaction)的ACID四大特性

	A:atomicity 原子性 - 不可分割
	C:consistency  一致性 - 事务前后数据状态一致
	I:isolation 隔离性 - 多个事务不能看到彼此的中间状态 
	D：duration 持久性 - 事务完成后数据要持久化

在出现并发事务访问数据的时候，数据库底层有锁机制来保护数据，但是通常我们在写
SQL的时候不用显示的书写锁操作
数据库会根据我们设定的事务隔离级别自动的数据加锁
查看事务的隔离级别:select @tx_isolation
设置事务隔离级别:set session transaction isolation level read committed

并发数据访问可能出现的5种问题
第1类丢失更新
第2类丢失更新:提交的事务，把别人的更新丢失了
脏读(读脏数据):一个事务读取到了另一个事务未尚未提交的数据(百万中奖诈骗)
不可重复读:一个事务在读取查询结果时发现其他事务更新了数据导致无法读取(时间段问题)
幻读:一个事务在执行查询时发现了被其他事务提交了新的数据(读到了新加的数据)

mysql事务隔离级别
	Read Uncommitted:读未提交 (脏读 不可重复读 幻读)
	Read Committed:读提交 (不可重复读 幻读)
	Repeatable Read:可重复第  (幻读)
	Serializable:串行化
### 5.补充知识
垂直扩展:只更换硬件(越到后面投入产出比越低)
水平扩展:增加机器数量,扩大节点数量

阿里巴巴 - 去IOE运动
​	去掉IBM小型机
​	去掉Oracle数据库
​	去掉EMC存储设备

正则表达式是模式匹配的工具



### 6.练习

```mysql
-- 创建SRS数据库
drop database if exists SRS;
create database SRS default charset utf8 collate utf8_bin;
-- utf8_bin_ci:这种排序方式不区分大小写

-- 切换到SRS数据库
use SRS;

-- 创建学院表
create table tb_college
(
collid int not null auto_increment comment '学院编号',
collname varchar(50) not null comment '学院名称',
collmaster varchar(20) not null comment '院长姓名',
collweb varchar(511) default '' comment '学院网站',
primary key (collid)
);

-- 添加唯一约束
alter table tb_college add constraint uni_college_collname unique (collname);

-- 创建学生表
create table tb_student
(
stuid int not null comment '学号',
sname varchar(20) not null comment '学生姓名',
gender bit default 1 comment '性别',
birth date not null comment '出生日期',
addr varchar(255) default '' comment '籍贯',
collid int not null comment '所属学院编号',
primary key (stuid)
);

-- 添加外键约束
alter table tb_student add constraint fk_student_collid foreign key (collid) references tb_college (collid);

-- 创建教师表
create table tb_teacher
(
teaid int not null comment '教师工号',
tname varchar(20) not null comment '教师姓名',
title varchar(10) default '' comment '职称',
collid int not null comment '所属学院编号'
);

-- 添加主键约束
alter table tb_teacher add constraint pk_teacher primary key (teaid);

-- 添加外键约束
alter table tb_teacher add constraint fk_teacher_collid foreign key (collid) references tb_college (collid);

-- 创建课程表
create table tb_course
(
couid int not null comment '课程编号',
cname varchar(50) not null comment '课程名称',
credit tinyint not null comment '学分',
teaid int not null comment '教师工号',
primary key (couid)
);

-- 添加外键约束
alter table tb_course add constraint fk_course_tid foreign key (teaid) references tb_teacher (teaid);

-- 创建学生选课表
create table tb_score
(
scid int not null auto_increment comment '选课编号',
sid int not null comment '学号',
cid int not null comment '课程编号',
seldate date comment '选课时间日期',
mark decimal(4,1) comment '考试成绩',
primary key (scid)
);

-- 添加外键约束
alter table tb_score add constraint fk_score_sid foreign key (sid) references tb_student (stuid);
alter table tb_score add constraint fk_score_cid foreign key (cid) references tb_course (couid);
-- 添加唯一约束
alter table tb_score add constraint uni_score_sid_cid unique (sid, cid);


-- 插入学院数据
insert into tb_college (collname, collmaster, collweb) values 
('计算机学院', '左冷禅', 'http://www.abc.com'),
('外国语学院', '岳不群', 'http://www.xyz.com'),
('经济管理学院', '风清扬', 'http://www.foo.com');

-- 插入学生数据
insert into tb_student (stuid, sname, gender, birth, addr, collid) values
(1001, '杨逍', 1, '1990-3-4', '四川成都', 1),
(1002, '任我行', 1, '1992-2-2', '湖南长沙', 1),
(1033, '王语嫣', 0, '1989-12-3', '四川成都', 1),
(1572, '岳不群', 1, '1993-7-19', '陕西咸阳', 1),
(1378, '纪嫣然', 0, '1995-8-12', '四川绵阳', 1),
(1954, '林平之', 1, '1994-9-20', '福建莆田', 1),
(2035, '东方不败', 1, '1988-6-30', null, 2),
(3011, '林震南', 1, '1985-12-12', '福建莆田', 3),
(3755, '项少龙', 1, '1993-1-25', null, 3),
(3923, '杨不悔', 0, '1985-4-17', '四川成都', 3);

-- 插入老师数据
insert into tb_teacher (teaid, tname, title, collid) values 
(1122, '张三丰', '教授', 1),
(1133, '宋远桥', '副教授', 1),
(1144, '杨逍', '副教授', 1),
(2255, '范遥', '副教授', 2),
(3366, '韦一笑', '讲师', 3);

-- 插入课程数据
insert into tb_course (couid, cname, credit, teaid) values 
(1111, 'Python程序设计', 3, 1122),
(2222, 'Web前端开发', 2, 1122),
(3333, '操作系统', 4, 1122),
(4444, '计算机网络', 2, 1133),
(5555, '编译原理', 4, 1144),
(6666, '算法和数据结构', 3, 1144),
(7777, '经贸法语', 3, 2255),
(8888, '成本会计', 2, 3366),
(9999, '审计学', 3, 3366);

-- 插入选课数据
insert into tb_score (sid, cid, seldate, mark) values 
(1001, 1111, '2017-09-01', 95),
(1001, 2222, '2017-09-01', 87.5),
(1001, 3333, '2017-09-01', 100),
(1001, 4444, '2018-09-03', null),
(1001, 6666, '2017-09-02', 100),
(1002, 1111, '2017-09-03', 65),
(1002, 5555, '2017-09-01', 42),
(1033, 1111, '2017-09-03', 92.5),
(1033, 4444, '2017-09-01', 78),
(1033, 5555, '2017-09-01', 82.5),
(1572, 1111, '2017-09-02', 78),
(1378, 1111, '2017-09-05', 82),
(1378, 7777, '2017-09-02', 65.5),
(2035, 7777, '2018-09-03', 88),
(2035, 9999, date(now()), null),
(3755, 1111, date(now()), null),
(3755, 8888, date(now()), null),
(3755, 9999, '2017-09-01', 92);

-- 查询所有学生信息
select * from tb_student;
-- 查询所有课程名称及学分(投影和别名)
select cname 课程名, credit 学分 from tb_course;
-- 查询所有女学生的姓名和出生日期(筛选)
select sname 姓名,birth 出生日期 from tb_student where gender=0;
-- 查询所有80后学生的姓名、性别和出生日期(筛选)
select sname 姓名, if(gender, '男', '女') 性别 from tb_student 
where birth between '1980-1-1' and '1989-12-31';
-- 查询姓”杨“的学生姓名和性别(模糊)
select sname 姓名,if(gender, '男', '女') 性别 from tb_student 
where sname like '杨%';
-- 查询姓”杨“名字两个字的学生姓名和性别(模糊)
select sname 性别, if(gender, '男', '女') 性别 from tb_student
where sname like '杨_';
-- 查询姓”杨“名字三个字的学生姓名和性别(模糊)
select sname 性别, if(gender, '男', '女') 性别 from tb_student
where sname like '杨__';
-- 查询名字中有”不“字或“嫣”字的学生的姓名(模糊)
select sname 性别, if(gender, '男', '女') 性别 from tb_student
where sname like '%不%' or sname like '%嫣%';
-- 查询没有录入家庭住址的学生姓名(空值)
select sname 姓名 from tb_student where addr is null or addr='';
-- 查询录入了家庭住址的学生姓名(空值)
select sname 姓名 from tb_student where addr is not null and addr<>'';
-- 查询学生选课的所有日期(去重)
select distinct seldate 选课日期 from tb_score;
-- 查询学生的家庭地址(去重)
select distinct addr 籍贯 from tb_student where addr is not null and addr<>'';
-- 查询学生的姓名和生日按年龄从大到小排列(排序)
select sname 姓名,birth 生日 from tb_student order by birth asc;
-- 查询男学生和生日从大到小排序(排序)
select sname 姓名,birth 生日 from tb_student where gender=1 order by birth asc;
-- 查询年龄最大的学生的出生日期(聚合函数)
select min(birth) 出生日期 from tb_student;
-- 查询年龄最小的学生的出生日期(聚合函数)
select max(birth) 出生日期 from tb_student;
-- 查询男女学生的人数(分组和聚合函数)
select if(gender,'男','女') 性别 ,count(gender) 人数 
from tb_student group by gender;
-- 查询课程编号为1111的课程的平均成绩(筛选和聚合函数)
select avg(mark) 平均成绩 from tb_score where cid=1111;
-- 查询学号为1001的学生所有课程的总成绩(筛选和聚合函数)
select sum(mark) 总成绩 from tb_score where sid=1001;
-- 查询学号为1001的学生所有课程的平均分(筛选和聚合函数)
-- 空值是不纳入计算的
select avg(mark) 平均分 from tb_score where sid=1001;
-- 查询每个学生的学号和平均成绩(分组和聚合函数)
select sid 学号,avg(mark) 平均分 from tb_score 
where mark is not null group by sid order by 平均分 desc;
-- 查询平均成绩大于等于90分的学生的学号和平均成绩
--  having是分组后的筛选
select sid 学号,avg(mark) 平均分 from tb_score 
group by sid having 平均分>=90; 
-- 查询年龄最大的学生的姓名(子查询)
-- 子查询:在一个查询中又用到了另外一个查询的结果
select sname 姓名, birth 生日 from tb_student where birth=
(select min(birth) from tb_student);

select sname 姓名, year(now())-year(birth) 年龄 from tb_student
where birth=(select min(birth) from tb_student);
-- 查询选了两门以上的课程的学生姓名(子查询/分组条件/集合运算)
select sname 姓名 from tb_student where stuid in
(select sid from tb_score group by sid having count(sid)>2);
-- 查询选课学生的姓名和平均成绩(子查询和连接查询)
-- 笛卡尔积 注意:在连接查询时如果没有连接条件就会形成笛卡尔积
select sname 姓名,平均成绩 from tb_student t1,
(select sid ,avg(mark) 平均成绩 from tb_score group by sid) t2 
where t1.stuid=t2.sid;

select sname,avgmark from tb_student inner join
(select sid,avg(mark) as avgmark from tb_score group by sid )t2
on stuid=sid;
-- 先放连表条件再放筛选条件
select sname,cname,mark 
from tb_score,tb_student,tb_course 
where stuid=sid and couid=cid and mark is not null;

select sname,cname mark from tb_student
inner join tb_score on stuid=sid
inner join tb_course on couid=cid
where mark is not null;

-- 查询学生姓名、所选课程名称和成绩(连接查询)
select sname,cname mark from tb_student
inner join tb_score on stuid=sid
inner join tb_course on couid=cid
where mark is not null;

-- 查询每个学生的姓名和选课数量(左外连接(left join)和子查询)
-- outer join 
-- 左外连接left join:把左表(写在前面的表)不满足连接条件的记录也查出来对应记录补null值

-- 右外连接right join:把右表(写在后面的表)不满足连接条件的记录也查出来对应记录补null值
select sname 姓名,number 选课数量 from tb_student left join
(select sid ,count(sid) as number from tb_score group by sid) t2
on stuid=sid;

select sname,total from tb_student,
(select sid,count(sid) as total from tb_score GROUP BY sid) tb_temp
where stuid=sid(+)

-- 标准sql在等号左边放+表示左外连接，+放等号右边表示右外连接
```

练习二
```mysql
-- 创建人力资源管理系统数据库
drop database if exists HRS;
create database HRS default charset utf8 collate utf8_bin;

-- 切换数据库上下文环境
use HRS;

-- 删除表
drop table if exists TbEmp;
drop table if exists TbDept;

-- 创建部门表
create table TbDept
(
dno tinyint not null comment '部门编号',
dname varchar(10) not null comment '部门名称',
dloc varchar(20) not null comment '部门所在地',
primary key (dno)
);

-- 添加部门记录
insert into TbDept values (10, '会计部', '北京');
insert into TbDept values (20, '研发部', '成都');
insert into TbDept values (30, '销售部', '重庆');
insert into TbDept values (40, '运维部', '深圳');

-- 创建员工表
create table TbEmp
(
eno int not null comment '员工编号',
ename varchar(20) not null comment '员工姓名',
job varchar(20) not null comment '员工职位',
mgr int comment '主管编号',
sal int not null comment '月薪',
comm int comment '月补贴',
dno tinyint comment '所在部门编号',
primary key (eno)
);

-- 添加外键约束
alter table TbEmp add constraint fk_dno foreign key (dno) references TbDept(dno) on delete set null on update cascade;
-- on delete set null on update casade:当删除主键时把它作为外键的值全部设置为null,当更新的时候，关联更新将其设置为外键的的列
-- on delete restrict on update restrict:不允许删除和更新主键列(外键关联的主键列)，默认情况下

-- 添加员工记录
insert into TbEmp values (7800, '张三丰', '总裁', null, 9000, 1200, 20);
insert into TbEmp values (2056, '乔峰', '分析师', 7800, 5000, 1500, 20);
insert into TbEmp values (3088, '李莫愁', '设计师', 2056, 3500, 800, 20);
insert into TbEmp values (3211, '张无忌', '程序员', 2056, 3200, null, 20);
insert into TbEmp values (3233, '丘处机', '程序员', 2056, 3400, null, 20);
insert into TbEmp values (3251, '张翠山', '程序员', 2056, 4000, null, 20);
insert into TbEmp values (5566, '宋远桥', '会计师', 7800, 4000, 1000, 10);
insert into TbEmp values (5234, '郭靖', '出纳', 5566, 2000, null, 10);
insert into TbEmp values (3344, '黄蓉', '销售主管', 7800, 3000, 800, 30);
insert into TbEmp values (1359, '胡一刀', '销售员', 3344, 1800, 200, 30);
insert into TbEmp values (4466, '苗人凤', '销售员', 3344, 2500, null, 30);
insert into TbEmp values (3244, '欧阳锋', '程序员', 3088, 3200, null, 20);
insert into TbEmp values (3577, '杨过', '会计', 5566, 2200, null, 10);
insert into TbEmp values (3588, '朱九真', '会计', 5566, 2500, null, 10);

-- 查询薪资最高的员工姓名和工资
select ename,sal+comm from TbEmp where sal+comm=(select max(sal+comm) from TbEmp);

-- 查询员工的姓名和年薪((月薪+补贴)*12)
select ename 姓名,12*(sal+ifnull(comm,0)) 年薪 from TbEmp;

select ename 姓名,12*(sal+ if(comm,comm,0)) 年薪 from TbEmp where 12*(sal+ if(comm,comm,0))>50000;
-- 查询有员工的部门的编号和人数
select dno 部门编号,count(dno) 人数 from TbEmp group by dno;

-- 查询所有部门的名称和人数,使用左外连接
select dname 部门,count(tbemp.dno) 人数 from TbEmp,TbDept 
where tbemp.dno=tbdept.dno(+) group by tbemp.dno;

select dname,ifnull(total,0) from tbdept t1 left join
(select dno,count(dno) as total from tbemp group by dno)t2
on t1.dno=t2.dno;

-- 查询薪资最高的员工(Boss除外)的姓名和工资
select ename as 姓名,sal+comm as 工资 from tbemp 
where sal+comm=(select max(sal+comm) from tbemp where mgr is not null);

-- 查询薪水超过平均薪水的员工的姓名和工资
select ename 姓名,sal as 工资 from tbemp 
where sal>(select avg(sal) from tbemp);

-- 查询薪水超过其所在部门平均薪水的员工的姓名、部门编号和工资
select ename, t1.dno,sal from tbemp t1,
(select dno,avg(sal)as avgsal from tbemp group by dno)t2
where t1.dno=t2.dno and sal>avgsal;

-- 查询部门中薪水最高的人姓名、工资和所在部门名称

select ename,sal as salary,dname from tbemp t1,
(select dno,dname from tbdept group by dno) t2,
(select dno,max(sal) maxsal from tbemp group by dno)t3
where t1.dno=t3.dno and sal=maxsal and t2.dno=t1.dno; 


-- 查询主管的姓名(去重)
select ename from tbemp,
(select mgr from tbemp where mgr is not null group by mgr) t2
where t2.mgr=tbemp.eno;


select ename,job from tbemp,
(select distinct mgr from tbemp where mgr is not null) t2
where t2.mgr=tbemp.eno;
-- 说明:去重操作和集合运算效率是非常低下的
-- 通常建议用exists和not exists操作来替代去重和集合运算
select ename,job from tbemp t1
where exists(select 'x' from tbemp t2 where t1.eno=t2.mgr);

```
### 7.python连接mysql

命令行创建虚拟环境: python -m venv venv