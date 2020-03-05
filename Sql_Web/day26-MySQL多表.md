---
typora-copy-images-to: img
---

# day16-MySQL多表

# 学习目标

- [ ] 能够使用内连接进行多表查询

- [ ] 能够使用左外连接和右外连接进行多表查询

- [ ] 能够使用子查询进行多表查询

- [ ] 能够理解事务的概念

- [ ] 能够说出事务的原理

- [ ] 能够在MySQL中使用事务

  

# 第一章-连接查询

## 环境准备

```sql
-- 创建一张分类表(类别id,类别名称.备注:类别id为主键并且自动增长)
CREATE TABLE t_category(
		cid INT PRIMARY KEY auto_increment,
		cname VARCHAR(40)
);
INSERT INTO t_category values(null,'手机数码');
INSERT INTO t_category values(null,'食物');


-- 创建一张商品表(商品id,商品名称,商品价格,商品数量,类别.备注:商品id为主键并且自动增长)

CREATE TABLE t_product(
		pid INT PRIMARY KEY auto_increment,
		pname VARCHAR(40),
		price DOUBLE,
		num INT,
		cno INT
);

insert into t_product values(null,'苹果电脑',18000,10,1);
insert into t_product values(null,'iPhone8s',5500,100,1);
insert into t_product values(null,'iPhone7',5000,100,1);
insert into t_product values(null,'iPhone6s',4500,1000,1);
insert into t_product values(null,'iPhone6',3800,200,1);
insert into t_product values(null,'iPhone5s',2000,10,1);
insert into t_product values(null,'iPhone4s',18000,1,1);

insert into t_product values(null,'方便面',4.5,1000,2);
insert into t_product values(null,'咖啡',10,100,2);
insert into t_product values(null,'矿泉水',2.5,100,2);

-- 给 商品表添加外键
ALTER TABLE t_product ADD FOREIGN KEY(cno) REFERENCES t_category(cid);
```

## 知识点-交叉查询【了解】

### 1.目标

+ 了解交叉查询

### 2.分析

​	交叉查询把若干张表(>=2)没有条件的连接在一起,进行展示

### 3.讲解

1. 语法

```sql
select a.列,a.列,b.列,b.列 from a,b ;  

select a.*,b.* from a,b ;  
--或者 
select * from a,b;
```

2. 练习: 使用交叉查询类别和商品

```
select * from t_category,t_product;
```

### 3.小结

- 交叉查询的结果不是我们想要的， 会产生错误信息。



## 知识点-内连接查询【重点】

### 1.目标

+ 掌握内连接查询

### 2.分析

​	交叉查询产生这样的结果并不是我们想要的，那么怎么去除错误的，不想要的记录呢，当然是通过条件过滤。通常要查询的多个表之间都存在关联关系，那么就通过==关联关系(主外键关系)==去除笛卡尔积。

==得到多张表之间共有的数据==

### 3.讲解

#### 3.1 隐式内连接

隐式里面是没有inner关键字的

```mysql
select [字段,字段,字段][*] from a,b where 连接条件 (a表里面的主键 = b表里面的外键) 
```

#### 3.2 显示内连接

显示里面是有inner关键字的

```mysql
select [字段,字段,字段][*] from a [inner] join b on 连接条件 [ where 其它条件]

select [字段,字段,字段][*] from a [inner] join b on 连接条件
join c on 连接条件
```

#### 3.3练习

查询所有类别下的商品信息,如果该类别下没有商品则不展示.

```mysql
-- 2.1 隐式内连接方式 
select *from t_category  c, t_product  p WHERE c.cid = p.cno;
-- 2.2 显示内连接方式  
SELECT * from t_category c INNER JOIN t_product p ON c.cid = p.cno
```

### 4.小结

- 内连接
  - 作用： 消除交叉查询的错误结果
  - 特征： 得到你有我有大家都有的公有数据
  - 隐式： `select ...   from  a,b  where 连接条件（a.字段=b.字段）`
  - 显示： `select ...   from  a join b  on 连接条件（a.字段=b.字段）`

## 知识点-外连接【重点】

### 1.目标

+ 掌握外连接查询

### 2.分析

​	我们发现内连接查询出来的是公共部分. 如果要保证某张表的全部数据情况下进行连接查询. 那么就要使用外连接查询了.  外连接分为左外连接和右外连接

### 3.讲解

#### 3.1左外连接

​	以join左边的表为主表,展示主表的所有数据,根据条件查询连接右边表的数据,若满足条件则展示,若不满足则以null显示.可以理解为：==在内连接的基础上保证左边表的数据全部显示==

1. 语法

```mysql
select 字段 from a left [outer] join b on 条件 [where 条件]
```

2. 练习:查询所有类别下的商品信息

```mysql
SELECT * FROM t_category c LEFT OUTER JOIN t_product p ON c.cid = p.cno
```

#### 3.2右外连接

​	以join右边的表为主表,展示右边表的所有数据,根据条件查询join左边表的数据,若满足则展示,若不满足则以null显示

可以理解为：在内连接的基础上保证右边表的数据全部显示

1. 语法

```mysql
select 字段 from a right [outer] join b on 条件
```

2. 练习:查询所有商品所对应的类别信息,要求种类全部展示

```mysql
SELECT * FROM t_category c RIGHT  OUTER JOIN t_product p ON c.cid = p.cno
```

### 4.小结

- 左外连接（主表是 join的左边）

  ```mysql
select *  from a left join b on 连接条件（a.字段=b.字段） 
  ```
  
  

- 右外连接（主表是 join的右边）

  ```mysql
  
  select *  from a right join b on 连接条件（a.字段=b.字段）
  ```



# 第二章-子查询【重点】

## 子查询入门

### 1.目标

+ 知道什么是子查询

### 2.分析

​	我们刚刚讲解了内连接和外连接查询, 但是如果遇到很复杂的场景, 内连接和外连接查询可能查询不出来.我们就可以使用子查询了.

### 3.讲解

#### 3.1什么是子查询  

直观一点: 一个查询里面至少包含2个select

+   一个查询的结果做为另一个查询的条件
+   有查询的嵌套，内部的查询称为子查询
+   子查询要使用圆括号

#### 3.2子查询初体验

eg:查询  价格高于IPhone5s的  商品信息

- 第一步:查询价格高iPhone5s的商品信息

```
SELECT * FROM t_product WHERE price > (第一步语句作为条件)
```

- 第二步:查询Iphone5s的商品价格

```
SELECT price FROM t_product where pname = 'iPhone5s';
```

+ 整个语法

```
SELECT * FROM t_product WHERE price > (SELECT price FROM t_product WHERE pname = 'iPhone5s')
```

### 3.小结

- 子查询
  - 至少2个select语句
  - 子查询用圆括号()括起来
  - 子查询可以放在查询后面做条件



## 知识点-子查询进阶

### 1.目标

+ 掌握子查询的三种类别

### 2.分析

​	尽管子查询的语法很灵活,没有固定的写法.但是它也有一些规律.

### 3.讲解

#### 3.1子查询的结果是一个值的情况

![img](img/tu_1.png)

1. 子查询结果只要是==单行单列==，肯定在WHERE后面作为条件，父查询使用：比较运算符，如：> 、<、<>、= 等

```
SELECT 查询字段 FROM 表 WHERE 列 =（子查询）;
```

2. 练习

   查询类别是手机数码的所有商品信息

```
 select *from product where cno =  (SELECT cid FROM category where cname ='手机数码');
```

​	查询和方便面是同一类别的商品信息,但是不包括方便面

```
 SELECT *from product where cno = (SELECT cno from Product where pname ='方便面') and pname <> '方便面';
```

#### 3.2子查询的结果是单列多行的情况  

![img](img/tu_2.png)

1. 子查询结果是单例多行，结果集类似于一个数组，父查询使用IN运算符

```
SELECT 查询字段 FROM 表 WHERE 列 IN （子查询）;
```

2. 练习

   查询类别属于手机数码和食物的商品信息

```
SELECT * FROM t_product WHERE cno in (SELECT cid FROM t_category WHERE cname in('手机数码','食物'));
```

​	

#### 3.3子查询的结果是多行多列的情况

![img](img/tu_3.png)

1. 子查询结果只要是多列，肯定在FROM后面作为表; 子查询作为表需要取别名，否则这张表没有名称则无法访问表中的字段

```
SELECT 查询字段 FROM （子查询） as 表别名 WHERE 条件;
```

2. 练习

   查询所属类别编号(cno)是1的商品里面的最高价格

```
SELECT MAX(c1.price) from (SELECT * FROM product where cno = 1) as c1
```

### 4.小结

- 子查询进阶三种情况
  - 结果集是一个值： `select ... from 表名 where = (子查询)`
  - 结果集是多个值： `select ... from 表名 where in (子查询)`
  - 结果集是多行多列。看做表： `select ... from (子查询) as 别名`

# 第三章-事务

## 环境的准备

```sql
-- 账户表
create table account(
    id int primary key auto_increment,
    name varchar(20),
    money double
);

insert into account values (null,'zs',1000);
insert into account values (null,'ls',1000);
insert into account values (null,'ww',1000);
```

## 知识点-事务的概述

### 1.目标

+ 掌握什么是事务以及事务作用

### 2.路径

1. 什么是事务
2. 事务作用

### 3.讲解

#### 3.1什么是事务 transaction

+ 事务指逻辑上的一组操作，组成这组操作的单元要么全部成功，要么全部失败。
+ 操作: zs给赵四进行转账100.
  + zs的钱就会减少100
  + 赵四加100

#### 3.2.事务的作用

​	保证一组操作全部成功或者失败。   

### 4.小结

- 事务的概念： 
  - 逻辑上的一组操作，要么全部成功，要么全部失败
- 事务的作用
  - 保障 逻辑上的一组操作，要么全部成功，要么全部失败



## 知识点-MYSQL进行事务管理 

### 1.目标

+ 掌握MYSQL进行事务管理 

### 2.路径

1. 自动事务(mysql默认)
2. 手动开启一个事务 
3. 事务原理
4. 事务回滚点

### 3.讲解

#### 3.1自动事务(mysql默认)

​	一条sql语句就是一个事务

#### 3.2手动开启一个事务 

+ 方式一: 手动开启事务的方式 【掌握】

  ​	start transaction;开启事务

  ​    commit；提交    	

  ​	 rollback；回滚

+ 方式二: 设置MYSQL中的自动提交的参数【了解】

  查看MYSQL中事务是否自动提交

  ```
  show variables like '%commit%';
  ```

  设置自动提交的参数为OFF

  ```
  set autocommit = 0;-- 0:OFF  1:ON
  ```

#### 3.3事务原理【了解】

​	事务开启之后, 所有的操作都会临时保存到事务日志中, 事务日志只有在得到==commit命令才会同步到数据表中==，其他任何情况都会清空事务日志(rollback，断开连接)   

1)    客户端连接数据库服务器，创建连接时创建此用户临时日志文件

2)    开启事务以后，所有的操作都会先写入到临时日志文件中

3)    所有的查询操作从表中查询，但会经过日志文件加工后才返回

4)    如果事务提交则将日志文件中的数据写到表中，否则清空日志文件。

#### 3.4回滚点

##### 3.4.1什么是回滚点 savepoint 保存点

​	在某些成功的操作完成之后，后续的操作有可能成功有可能失败，但是不管成功还是失败，前面操作都已经成功，可以在当前成功的位置设置一个回滚点。可以供后续失败操作返回到该位置，而不是返回所有操作，这个点称之为回滚点。

##### 3.4.2  回滚点的操作语句

![img](img/tu_5.png)

##### 3.4.3具体操作

1)    将数据还原到1000

2)    开启事务

3)    让张三账号减3次钱 

4)    设置回滚点：savepoint three_times;

5)    让张三账号减4次钱

6)    回到回滚点：rollback to three_times;

+ 总结：设置回滚点可以让我们在失败的时候回到回滚点，而不是回到事务开启的时候。

##### 3.4.4应用场景

 插入大量的数据的时候.  1亿条数据 需要插入很久.  要求: 1亿条数据是一个整体,要么全部插入成功的 要么都不插入成功.

### 4.小结

- mysql管理事务
  - 开启事务： `start transaction`
  - 提交： `commit`
  - 回滚： `rollback`
- 回滚点
  - 设置： savepoint 名字自定义
  - 使用： rollback to 回滚点的名字



## 知识点-事务特性和隔离级别

### 1.目标

+ 掌握事务的特性和隔离级别

### 2.路径

1. 事务特性
2. 事务隔离级别

### 2.讲解

#### 2.1事务特性【面试题】ACID

- 原子性（Atomicity）原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。   
  + zs给ls转账100
    + zs减少100
    + ls增加100
- 一致性（Consistency）事务前后数据的完整性必须保持一致.   
  - zs给ls转账100; 2000;
    - zs减少100,ls增加100  ;  2000
    - 2000
- 持久性（Durability）持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响。  
  + 

- 隔离性（Isolation）事务的隔离性是指多个用户并发操作数据库时，一个用户的事务不能被其它用户的事务所干扰，多个并发事务之间数据要相互隔离。 简单来说: 事务之间互不干扰


#### 2.2如果不考虑隔离性，会引发下面的问题

​	事务在操作时的理想状态： 所有的事务之间保持隔离，互不影响。因为并发操作，多个用户同时访问同一个数据。可能引发并发访问的问题

![img](img/tu_6.png)

#### 2.3事务隔离级别

​	可以通过设置事务隔离级别解决读的问题

##### 2.3.1事务四个隔离级别  

| **级别** | **名字** | **隔离级别**         | **脏读** | **不可重复读** | **幻读** | **数据库默认隔离级别** |
| ------ | ------ | ---------------- | ------ | --------- | ------ | ------------- |
| **1**  | 读未提交   | read uncommitted | 是      | 是         | 是      |               |
| **2**  | 读已提交   | read committed   | 否      | 是         | 是      | Oracle        |
| **3**  | 可重复读   | repeatable read  | 否      | 否         | 是      | MySQL         |
| **4**  | 串行化    | serializable     | 否      | 否         | 否      |               |

> 隔离级别越高，性能(效率)越差，安全性越高。

##### 2.3.2设置隔离级别

- 设置事务隔离级别

```mysql
set session transaction isolation level  隔离级别;
eg: 设置事务隔离级别为read uncommitted
set session transaction isolation level read uncommitted;
```

- 查询当前事务隔离级别

```
select @@tx_isolation;
```

### 3.小结

- 事务特性
  - ACID
  - A： 原子性； 
  - C： 一致性；
  - I： 隔离性；
  - D: 持久性；
- 问题
  - 脏读： 一个事务读到了另一个事物还没有提交的数据
  - 不可重复读： 一个事务内，多次读到的数据的==内容==不一致
  - 幻读：一个事务内，多次读到的数据的==数量==不一致
- 隔离级别
  - 读未提交： 脏读、不可重复读、幻读
  - 读已提交：不可重复读、幻读
  - 可重复读： 幻读；  mysql默认的隔离级别
  - 串行化： 性能最差 



## 实操-演示数据库安全性问题的发生

### 1.目标

+ 能够演示数据库安全性问题的发生
+ ==不要求练习； 只是为了更好地理解“不考虑隔离性的问题、隔离级别”==

### 2.路径

1. 演示脏读
2. 演示不可以重复读
3. 演示避免不可重复读
4. 演示隔离级别Serializable

### 3.讲解

![1555812385275](img/1555812385275.png) 

### 4.小结

# 第四章-创建用户和授权(DCL)【了解】

## 知识点-创建用户

### 1.目标

+ 了解使用root账号创建用户

### 2.分析

​	我们现在默认使用的都是root用户，超级管理员，拥有全部的权限。但是，一个公司里面的数据库服务器上面可能同时运行着很多个项目的数据库。所以，我们应该可以根据不同的项目建立不同的用户，分配不同的权限来管理和维护数据库。

​	我们刚进公司的时候, 一般是不可能拥有root账户的, 一般都是一些普通用户. 这个章节的知识点都是基于root账户操作的.

### 3.讲解

#### 3.1创建用户

1. 语法：  =='用户名'==@=='主机名'==

```
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
```

2. 练习

```mysql
create user 'zs'@'localhost' identified by '123456';
create user 'ls'@'localhost' identified by '123456';
create user 'ww'@'localhost' identified by '123456';
```

![img](img/tu_7.png)

#### 3.2删除用户

+ 用户

```
DROP USER '用户名'@'主机名';
```

+ 删除zs用户

```mysql
drop user 'zs'@'localhost';
```



## 知识点-授权

### 1.目标

+ 了解使用root账号进行授权

### 2.讲解

#### 2.1授权

1. 语法

```
GRANT 权限1, 权限2... ON 数据库名.表名 TO '用户名'@'主机名';
```

![img](img/tu_8.png)

2. 练习

   给zl用户分配对web16这个数据库操作的权限：创建表，修改表，插入记录，更新记录，查询

```
grant create,alter,insert,update,select on day16.* to 'zl'@'localhost';
```

​	给zl用户分配所有权限，对所有数据库的所有表 

```
grant all on *.* to 'zl'@'localhost';
```



#### 2.2查看权限

1. 语法

```
SHOW GRANTS FOR '用户名'@'主机名';
```

2. 练习

   查询查看user1用户的权限

```
SHOW GRANTS FOR 'zs'@'主机名';
```

#### 2.3撤销授权

1. 语法

```
REVOKE  权限1, 权限2... ON 数据库.表名 from '用户名'@'主机名'; 
或者
revoke all on test.* from '用户名'@'主机名'; 
```

![img](img/tu_9.png)

2. 练习

   撤销zs用户对test数据库所有表的操作的权限

```
revoke all on test.* from 'zs'@'localhost';
```

#### 2.4.修改普通用户的密码

1. 语法

```
set password for '用户名'@'主机名' = password('新密码');
```

2. 练习   

将'zs'@'localhost'的密码改成'666666'

```
set password for 'zs'@'localhost' = password('666666');
```

# 总结

- 内连接
  - 特征： 获得双方都有的数据； 解决交叉查询的错误结果
  - 隐式： `select ..  from a,b where 连接条件（a.字段=b.字段）`
  - 显示: `select ... from a join b on 连接条件（a.字段=b.字段）`
- 外连接
  - 左外连接（主表： join的左边表）
    - `select ... from a left join b on 连接条件（a.字段=b.字段）`
  - 右外连接（主表： join右边的表）
    - `select ... from a right join b on 连接条件（a.字段=b.字段）`

- 子查询
  - 特征
    - 至少2条select语句
    - 子查询可以作为另一个查询的where条件
    - 子查询需要使用()括起来
  - 结果集是一个值
    - select ... from 表名 where 列= (子查询)
  - 结果集是多个值
    - select ... from 表名 where 列  in (子查询)
  - 结果集是多行多列，看成表，接在from后面
    - select ... from (子查询) ==as 别名==
- 事务
  - 概念： 组成逻辑单元的每个操作，要么完全成功要么完全失败
  - 作用： 保证每个操作单元要么完全成功要么完全失败
  - 事务特性
    - ACID
    - A： 原子性
    - C： 一致性
    - I： 隔离性
    - D： 持久性
  - 问题
    - 脏读： 一个事务读到了另一个事务尚未提交的数据
    - 不可重复读： 一个事务里面，多次读取数据==内容==不一致
    - 幻读：  一个事务里面，多次读取数据==数量==不一致
  - 隔离级别
    - 读未提交： 脏读、不可重复读、幻读
    - 读已提交： 不可重复读、幻读；  oracle默认
    - 可重复读： 幻读  ； mysql默认
    - 串行化： 解决任何问题，性能最差