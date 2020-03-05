---
typora-copy-images-to: img
---

# day15-MySQL进阶

# 学习目标

- [ ] 能够使用SQL语句进行排序

- [ ] 能够使用聚合函数

- [ ] 能够使用SQL语句进行分组查询

- [ ] 能够完成数据的备份和恢复

- [ ] 能够说出多表之间的关系及其建表原则

- [ ] 能够理解数据库的三大范式 概念

  

# 第一章-可视化软件的使用

## 实操-可视化软件的使用【会用就OK】

### 1.目标

+ 掌握可视化软件的使用

### 2.步骤

+ 连接数据库
+ 对数据库的操作
+ 对表的操作
+ 对数据的操作

### 3.讲解

- 连接数据库

  ![1536021958539](img/1-connectDb.gif)


- 对数据库的操作

  ![img](img/2-operateDb.gif)


- 对表的操作


  - 创建表

    ![1556001707105](img/3-createTb.png) 

    ![1556001766769](img/4-addColumn.png)

  + 修改表

     ![1556001813064](img/5-alterTb.png)

    ![1556001844616](img/6-alterTb2.png)

  + 删除表

     ![1556001871726](img/7-delTb.png)

- 对数据的操作

  + 插入数据

    ![1556001982818](img/8-insertData.png) 

  + 删除数据

    ![1556001999671](img/9-delData.png) 

  + 修改数据
  
    直接选中数据行，输入新的内容即可。

### 4.小结

1. SQLyog工具的使用
2. 掌握对数据库、表、行记录的操作即可

# 第二章-查询记录【重点】

## 环境准备

```mysql
# 创建学生表(有sid,学生姓名,学生性别,学生年龄,分数列,其中sid为主键自动增长)
CREATE TABLE student(
	sid INT PRIMARY KEY auto_increment,
	sname VARCHAR(40),
	sex VARCHAR(10),
	age INT,
  score DOUBLE(5,2)
);

INSERT INTO student VALUES(null,'zs','男',18,98.5);
INSERT INTO student VALUES(null,'ls','女',18,96.5);
INSERT INTO student VALUES(null,'ww','男',15,50.5);
INSERT INTO student VALUES(null,'zl','女',20,98.5);
INSERT INTO student VALUES(null,'tq','男',18,60.5);
INSERT INTO student VALUES(null,'wb','男',38,98.5);
INSERT INTO student VALUES(null,'小丽','男',18,100);
INSERT INTO student VALUES(null,'小红','女',28,28);
INSERT INTO student VALUES(null,'小强','男',21,95);
```

## 知识点-排序查询

### 1.目标

+ 能够使用SQL语句进行排序

### 2.分析

​	有时候我们需要对查询出来的结果排序显示，那么就可以通过`ORDER BY`子句将查询出的结果进行排序。排序可以根据一个字段排，也可以根据多个字段排序，排序只是对查询的结果集排序，并不会影响表中数据的顺序。

### 3.讲解

#### 3.1单列排序

1. 语法:  只按某一个字段进行排序，单列排序

```mysql
SELECT 字段名 FROM 表名 [WHERE 条件] ORDER BY 字段名 [ASC|DESC];  //ASC: 升序，默认值; DESC: 降序
```

2. 练习: 以分数降序查询所有的学生

```mysql
SELECT * FROM student ORDER BY score DESC
```

#### 3.2组合排序

1. 语法: 同时对多个字段进行排序，如果第1个字段相等，则按第2个字段排序，依次类推

```mysql
SELECT 字段名 FROM 表名 WHERE 字段=值 ORDER BY 字段名1 [ASC|DESC], 字段名2 [ASC|DESC];
-- order by 出现一次
-- 多个字段,分隔开
```

2. 练习: 以分数降序查询所有的学生, 如果分数一致,再以age降序

```
SELECT * FROM student ORDER BY score DESC, age DESC
```

	3. 练习： 以分数降序查询所有的男生，分数一致则再以age降序排序

```mysql
select * from student where sex='男生' order by score desc,age desc;
```



### 4.小结

- 语法

  - 关键词：  ==order by 列 [ASC |  DESC]==

  - 完整语法

    ```mysql
    select ...   from 表名  [where 条件]  order by 列 [ASC|DESC]  -- 单列排序
    select ...   from 表名  [where 条件]  order by 列 [ASC|DESC], 列 [ASC|DESC]  -- 多列、组合排序
    ```
  
  

## 知识点-聚合函数

### 1.目标

+ 能够使用聚合函数

### 2.分析

​	之前我们做的查询都是横向查询，它们都是根据条件一行一行的进行判断，而使用聚合函数查询是==纵向查询==，它是对一列的值进行计算，然后返回一个结果值。聚合函数会忽略空值NULL

### 3.讲解

| 聚合函数          | 作用          |
| ------------- | ----------- |
| max(列名)       | 求这一列的最大值    |
| min(列名)       | 求这一列的最小值    |
| avg(列名)       | 求这一列的平均值    |
| **count(列名)** | 统计这一列有多少条记录 |
| sum(列名)       | 对这一列求总和     |

1. 语法

```sql
SELECT 聚合函数(列名) FROM 表名 [where 条件];
```

2. 练习

- 求出学生表里面的最高分数
-  求出学生表里面的最低分数
- 求出学生表里面的分数的总和
- 求出学生表里面的平均分
- 统计学生的总人数 


```mysql
-- 求出学生表里面的最高分数
SELECT MAX(score) FROM student
-- 求出学生表里面的最低分数
SELECT MIN(score) FROM student
-- 求出学生表里面的分数的总和(忽略null值)
SELECT SUM(score) FROM student
-- 求出学生表里面的平均分
SELECT AVG(score) FROM student
-- 统计学生的总人数 (忽略null) 
SELECT COUNT(sid) FROM student
SELECT COUNT(*) FROM student
```

> 注意:  聚合函数会忽略空值NULL

​	我们发现对于NULL的记录不会统计，建议如果统计个数则不要使用有可能为null的列，但如果需要把NULL也统计进去呢？我们可以通过 IFNULL(列名，默认值) 函数来解决这个问题. 如果列不为空，返回这列的值。如果为NULL，则返回默认值。 

聚合函数不会统计null值得这一行；null值对于聚合函数是无效的。

### 4.小结

- 聚合函数：
  - 最大值： `max(列)`
  - 最小值： `min(列)`
  - 总和： `sum(列)`
  - 平均值： `avg(列)`
  - 个数： `count(列使用主键列)`

## 知识点-分组查询

### 1.目标

+ 能够使用SQL语句进行分组查询

### 2.分析

​	 分组查询是指使用 ```GROUP BY```语句对查询信息进行分组

​	 GROUP BY怎么分组的？ 将分组字段结果中相同内容作为一组，如按性别将学生分成两组

​	 GROUP BY将分组字段结果中相同内容作为一组，并且返回每组的第一条数据，所以单独分组没什么用处。分组的目的就是为了统计，一般分组会跟聚合函数一起使用

### 3.讲解

#### 3.1分组

1. 语法

```mysql
SELECT 字段1,字段2... FROM 表名  [where 条件] GROUP BY 列
```

2. 练习:根据性别分组, 统计每一组学生的总人数

```mysql
-- 根据性别分组, 统计每一组学生的总人数
SELECT sex, count(*) FROM student GROUP BY sex
```

#### 3.2 分组后筛选 having

```mysql
SELECT 字段1,字段2... FROM 表名  [where 条件] GROUP BY 列 having 条件
```



+ 练习根据性别分组, 统计每一组学生的总人数> 5的(分组后筛选)

  ```mysql
  SELECT sex, count(*) FROM student GROUP BY sex HAVING count(*) > 5
  ```

+ 表数据如下：

  ![](img\1565838779980.png)

  Orders表数据如下所示，执行如下SQL语句，运行结果是？  

  ```mysql
  SELECT sex, COUNT(sid)  FROM student GROUP BY sex  where COUNT(sid) > 5; 
  
  SELECT sex, COUNT(sid)  FROM student where sex='女'  GROUP BY sex; 
  错误？正确？
  ```

![img](img/tu_1.png)

### 4.小结

- 分组查询

  - 关键词： `group by  列`

  - 完整的分组查询语法

    ```mysql
    select 聚合函数(列) from 表名 [where 条件] group by 列  
    
    select 聚合函数(列) from 表名 [where 条件] group by 列  having 条件
    ```
  ```
  
    
  
    
  ```

## 知识点-分页查询

### 1.目标

+ 掌握分页查询

### 2.分析

​	LIMIT是限制的意思，所以LIMIT的作用就是限制查询记录的条数. 经常用来做分页查询

### 3.讲解	

1. 语法

```mysql
select ... from ....[where 条件] limit a ,b
```

![img](img/tu_3.png)



​	b: 一页显示的数量(根据需求自定义的, 已知的)

​	a: 从哪里开始查询(从0开始计数) ;



2. 练习

   1. 假设每一页的大小是3，来做3个练习

   - 查询第一页的数据  
   - 查询第二页的数据
   - 查询第三页的数据

3. 归纳公式算法

```mysql

# 第一页
SELECT * FROM student LIMIT 0, 3;

# 第二页
SELECT * FROM student LIMIT 3, 3;

# 第三页
SELECT * FROM student LIMIT 6, 3;
```



==新的需求： 假设学生表有一百万的数据，每一页7条数据；现在我需要获取第800页的数据、甚至需要获取第1024页的数据。==，如何写出目标SQL？

分析一下：

```
-- 分页查询学生表  
-- 第1页  
a = 0, b=3   
-- 第2页
a = 3, b=3
-- 第3页
a = 6, b = 3
```



每页的大小： pageSize : 3
页码：  page

```
b = pageSize ;

a = (page-1)*pageSize 
```

某一页的条件：  a = (page -1) * pageSize, b= pageSize

得到分页公式：

```mysql
limit  (page -1) * pageSize, pageSize
```



- 练习： 
- 一页为4条记录，查询第二页的算法
  - limit 4, 4
- 学生表一百万的数据，每页7条数据，查询第800页的算法、第1024页的算法
  - limit (800-1)*7, 7；
  - limit(1024-1)*7,7;



### 4.小结

- 分页查询
  
  - 关键词：  limit 
    
  - 语法
    
    ```mysql
    select ...   from 表名 limit a,b;
    b:  指的是页的大小；页的容量
    a： 该页的起始行数； 第一行从0开始
    
    分页算法：
    页的大小；页的容量---> pageSize
    页码： page
    b: pageSize
    a: pageSize*(page-1)
    ```
    
    



## 查询的语法小结

```mysql
-- 基础查询
select [*] [列] [列 as 别名] [distinct 列] from 表名
-- 条件查询
select [*] [列] [列 as 别名] [distinct 列] from 表名 where 条件

-- 排序查询
select [*] [列] [列 as 别名] [distinct 列] from 表名 order by 列 desc;
-- 分组查询
select 聚合函数(列) [distinct 列] from 表名 where 条件 group by 列
-- 分页查询
select [*] [列] [列 as 别名] [distinct 列] from 表名 where 条件 limit a, b



-- 分组之后再分页(分页永远在最后面)
select ... from 表名 group by 字段 limit a,b;

-- -- 分页一定放在最后面
SELECT sex, COUNT(*) FROM student GROUP BY sex  ORDER BY COUNT(*) LIMIT 0,3 ;

-- where --》 group by --> having --> order by --> limit
SELECT sex, COUNT(*) FROM student WHERE  1=1  GROUP BY sex HAVING COUNT(*) > 5 ORDER BY COUNT(*)  LIMIT 0,3 ;

```



# 第三章-数据的备份和还原【了解】

## 实操-数据的备份和还原 

### 1.目标

- 掌握数据的备份和还原 

### 2.分析

​	在服务器进行数据传输、数据存储和数据交换，就有可能产生数据故障。比如发生意外停机或存储介质损坏。这时，如果没有采取数据备份和数据恢复手段与措施，就会导致数据的丢失，造成的损失是无法弥补与估量的。

### 3.讲解

#### 3.1命令行方式

1. 备份格式（命令行执行）

```mysql
mysqldump -u 用户名 -p 数据库 > 文件的路径   
Enter password: ****
mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名
```

2. 还原格式

```mysql
SOURCE 导入文件的路径;
```

> 注意：还原的时候需要先登录MySQL,并选中对应的数据库



练习： 

需要修改mysql安装目录下的配置文件my.ini 或者 my.cnf 文件客户端字符编码为gbk；

重启mysql服务；

打开命令行： windows徽标键+R，输入cmd 打开命令行。

备份执行： mysqldump -u 用户名 -p 数据库 > 文件的路径

此时可以在可视化工具删除该数据库下的表数据信息。

还原执行：

​	登录mysql服务，并选择使用数据库；

​	再执行 ： SOURCE 文件的路径

![1572087438688](img\1572087438688.png)



![1572086186875](img\1572086186875.png)

#### 3.2.使用工具备份和还原

1. 备份

![](img\1565864546879.png) 

2. 还原

![](img\1565864577673.png) 

### 4.小结

1. 会使用SqlYog、navicat可视化工具操作就行 了, 工作里面一般是运维在处理, 运维处理的话一般定时自动备份.



# 第四章-多表间的关系

## 知识点-为什么要拆表?

### 1.目标

+ 知道拆表原因

### 2.分析

1. 我们有一张产品表，这张表存放了产品信息、以及所属分类信息。

### 3.讲解

```mysql
CREATE TABLE product(
	pid INT PRIMARY KEY AUTO_INCREMENT,
	pname VARCHAR(40),
	price DOUBLE,
	num INT,
	cname VARCHAR(40)    -- 分类名称
);
INSERT INTO product VALUES(NULL,'华为手机',3000,10,'电子产品');
INSERT INTO product VALUES(NULL,'iPhonex',8000,20,'电子产品');
INSERT INTO product VALUES(NULL,'iPhone8s',5000,100,'电子产品');
INSERT INTO product VALUES(NULL,'iPhone7',4000,100,'电子产品');
INSERT INTO product VALUES(NULL,'iPhone6s',3000,1000,'电子产品');
INSERT INTO product VALUES(NULL,'iPhone5s',2000,5,'电子产品');

INSERT INTO product VALUES(NULL,'方便面',5,100,'食物');
INSERT INTO product VALUES(NULL,'咖啡',10,1000,'食物');
INSERT INTO product VALUES(NULL,'矿泉水',2.5,10,'食物');
```



#### 为什么要拆表？

- ==问题1==：产品信息、分类信息放一块，维护不方便
  - 问题：现在产品表存在100万条的数据，现在业务变更，需要将原来的'电子产品'改为'3C'
- ==问题2==：新增类别时，怎么办？



==拆表==：  让信息归属单一（一张表就表示一个实体）、便于维护、修改信息（在目标表修改即可，不影响其他的相关表）



1. 拆表工作 

```sql
-- 创建一张分类表(类别id,类别名称.备注:类别id为主键并且自动增长)
CREATE TABLE t_category(
	cid INT PRIMARY KEY auto_increment,
	cname VARCHAR(20)
);

-- 创建一张商品表(商品id,商品名称,商品价格,商品数量,类别编号.备注:商品id为主键并且自动增长)
CREATE TABLE t_product(
	pid int PRIMARY KEY auto_increment,
	pname VARCHAR(40),
	price DOUBLE,
	num INT,
	cno INT 
);

INSERT INTO t_product VALUES(NULL,'华为手机',3000,10,1);
INSERT INTO t_product VALUES(NULL,'iPhonex',8000,20,1);
INSERT INTO t_product VALUES(NULL,'iPhone8s',5000,100,1);
INSERT INTO t_product VALUES(NULL,'iPhone7',4000,100,1);
INSERT INTO t_product VALUES(NULL,'iPhone6s',3000,1000,1);
INSERT INTO t_product VALUES(NULL,'iPhone5s',2000,5,1);

INSERT INTO t_product VALUES(NULL,'方便面',5,100,2);
INSERT INTO t_product VALUES(NULL,'咖啡',10,1000,2);
INSERT INTO t_product VALUES(NULL,'矿泉水',2.5,10,2);


INSERT INTO t_category VALUES(NULL,'手机数码');
INSERT INTO t_category VALUES(NULL,'食物');

```

### 4.小结

- 拆表原因
  - 数据修改困难
  - 新增类别
- 经验法则：一张表（table）表示一个实体（现实中的实体： 用户、产品）





==拆表后还有问题么？==

- 数据可以随意修改，不受约束







## 知识点-外键约束

### 1.目标

+ 掌握外键的使用

### 2.分析

​	表和表之间存在一种关系，但是这个关系需要谁来维护和约束？

### 3.讲解

#### 3.1外键约束作用

​	保证数据引用完整性.  用来维护多表间关系

#### 3.2外键的语法

1. 添加外键

```sql
alter table 表 add  foreign key(列) references 表（主键列);
或者
alter table 表 add [CONSTRAINT] [外键名称] foreign key(字段) references 表（字段);
```

2. ​	**外键要求**：

   ==外键列的类型一定要和被参照列的类型一致==

   ==被参照的列(主键)必须为主键==

3. 给商品表添加外键

   1. `ALTER table t_product add foreign key(cno) references t_category(cid);`

4. 添加外键后的效果展示

   1. 不能随意删除类别信息了
   2. 不能乱增加产品信息了

> 注意:

​	==外键可以为null==

#### 3.3外键的练习

1. 给商品表添加外键

```mysql
ALTER table t_product add foreign key(cno) references t_category(cid);
```

2. 外键练习

| sid  | name | sex  |
| ---- | ---- | ---- |
| 1    | 张三 | 男   |
| 2    | 李四 | 女   |

| id   | score | sid1 |
| ---- | ----- | ---- |
| 1    | 99.5  | 2    |
| 2    | 100   | 1    |



```mysql
-- 学生表
CREATE TABLE t_student(
	sid int primary key,
	name varchar(50) not null,
	sex varchar(10)
);
	
-- 分数表
create table t_score( 
	id int primary key,
	score double,
	sid1 int
);

```

- 添加外键练习：

  ```mysql
  ALTER TABLE t_score ADD  FOREIGN KEY(id) REFERENCES t_student(sid);
  ```




#### 3.5 外键的级联【了解】

删除一方的时候另外的一方的数据没有任何用途的时候才可以使用 eg: 公司和员工

+ 要把类别的id值1，改成5，能不能直接修改呢？

```sql
UPDATE categoey SET cid=5 WHERE cid=2;
```

​	不能直接修改:Cannot delete or update a parent row: a foreign key constraint fails 如果副表(商品表)中有引用的数据,不能直接修改主表(类别表)主键

+ 要删除类别为1的类别,能不能直接删除呢？

```sql
DELETE FROM categoey WHERE id = 1;
```

​	不能直接删除:Cannot delete or update a parent row: a foreign key constraint fails 如果副表(商品表)中有引用的数据,不能直接删除主表(类别表)数据

+ 什么是级联操作

  在修改和删除主表的主键时，同时更新或删除副表的外键值，称为级联操作
  `ON UPDATE CASCADE` -- 级联更新，主键发生更新时，外键也会更新
  `ON DELETE CASCADE` -- 级联删除，主键发生删除时，外键也会删除

具体操作：

- 删除product表
- 重新创建product表，添加级联更新和级联删除

```sql
ALTER TABLE product ADD FOREIGN KEY(cno) REFERENCES category(cid) ON UPDATE CASCADE ON DELETE CASCADE
```



### 4.小结	

- 外键约束
  
  - 作用： 保障数据的完整性，维护表之间的关系
  
  - 语法
  
    ```mysql
    alter table 表名 add foreign key(列) references 表(主键)
    ```
  
  - 级联
  
    ```mysql
    alter table 表名 add foreign key(列) references 表(主键) on update CASCADE on delete cascade
    ```
  
    

## 知识点-多表间关系

### 1.目标

+ 能够说出多表之间的关系及其建表原则

### 2.分析

​	 设计数据库表 大部分都是从生活,习惯里面来的. eg: 下订单(orders)---> 谁下(user), 买了什么(product) 

​	现实生活中，实体与实体之间肯定是有关系的，比如：丈夫和妻子，部门和员工，学生与课程等。

​    那么我们在设计表的时候，就应该体现出表与表之间的这种关系！分成三种：

1. 一对多（多对一）：部门--员工 
2. 多对多：学生--课程
3. 一对一：一夫一妻

### 3.讲解

#### 3.1 一对多【重点】 

eg: 类别和商品, 用户和订单, 省和城市, 国家和省, 皇帝和妃子

- 一对多的关系：在**多**的一方建立外键去引用一的一方的主键

==产品、种类：==

![](img\1565939548936.png)

```sql
alter table product add foreign key(cid) references  category(cid)
```



#### 3.2 多对多【重点】  

eg: 学生和课程,   讲师和学生

- 多对多关系：增加一张中间表，存储的都是外键，一列指向多对多关系的一方的主键，另外一列指向另外一方的主键

==选课案例：==

![](img\1565939470363.png)

```mysql
alter table s_c_table add foreign key(sno) references student(sid);
alter table s_c_table add foreign key(cno) references cource(cid);
```

#### 3.3一对一【了解】

其实一张表里面就是1对1, 我们可以就使用一张表. 也可以拆表. eg:用户信息

- 一对一的关系：一张表可以存储

- 一对一：多张表表示，引入外键---》理解成一对多的关系（在证件号增加唯一约束）
  - 使用多张表： 一张表存储基本信息； 另外的表存放特定的信息： 履历、社会关系

==用户信息：==

![1565939341096](img\1565939341096.png)

### 4.小结

- 多表的关系
  - 1对多
    - 场景：一夫多妻
    - 建表原则： 在多的一方创建外键，去引用少的一方的主键
  - 多对多
    - 场景：选课
    - 建表原则：创建一张中间表，存储的都是外键,引用其他表的主键
  - 1对1
    - 场景：人--证件号码 
    - 建表原则： 一张表； 多张表：看成一对多的关系，使用外键关联



# 第五章-数据库设计三大范式

## 知识点-数据库设计三大范式

### 1.目标

+ 能够理解三大范式

### 2.分析

​	好的数据库设计对数据的存储性能和后期的程序开发，都会产生重要的影响。

​	建立数据库就需要满足一些规则来优化数据的设计和存储，这些规则就称为范式(NF： Normal Form)。

### 3.讲解

#### 3.1 1NF

##### 3.1.1概述

​	==数据库表的每一列都是不可分割的原子数据项，不能是集合、数组等非原子数据项==。

1NF要求：

- ==每一列不可再拆分成更细的粒度==

##### 3.1.2 应用

不符合1NF(如果不遵守第一范式，查询出数据还需要进一步处理（查询不方便）。)

![](img\1570525463879.png)

符合1NF

![](img\1570525481003.png)

##### 3.1.3 总结

​	如果不遵守第一范式，查询出数据还需要进一步处理（查询不方便）。

​	遵守第一范式，需要什么字段的数据就查询什么数据（方便查询）

#### 3.2 2NF

##### 3.2.1概述

2NF的特征：

- 在满足1NF的前提下，表中的每一个字段都完全依赖于主键。
- 一张表只做一件事

不符合2NF的现象：

		- 比如有一个主键有两个列，一个字段只依赖于主键中的其中一个列，这就是不符合第二范式。

##### 3.2.2 应用

不符合2NF

![](img\1570525505459.png)

拆表，每张表描述一件事情，符合2NF

![1566026086887](img\1570525530411.png)

##### 3.3.3总结

​	第二范式要求完全依赖主键，一张表只做一件事情

#### 3.3  3NF

##### 3.3.1概述

3NF的要求：

- 一个表中不包含其他表的非主键信息

==**每一列不能依赖于其它表的非主键列**==， 不允许出现传递性依赖

##### 3.3.2应用

不符合3NF，依赖于其他表的非主属性，产生了传递依赖，修改数据的时候多张表都需要修改（不方便修改）。

![1566026120673](img\1570525662881.png)

符合3NF

![1566026147984](img\1570525676944.png)

##### 3.2.3总结

​	如果不遵守第三范式，修改数据的时候多张表都需要修改（不方便修改）。

​	回过头来看一下，其实我们自己的产品表、种类表已经符合这三大范式了。

### 4.小结

- 三大范式
  - 1NF：表中的每一列都不可再拆分更细粒度
  - 2NF：满足1NF前提下，所有列必须完全依赖主键，一张表仅仅描述一个实体
  - 3NF： 满足2NF前提下，不允许出现传递性依赖，要求不能依赖另一个表的非主键列

# 总结

- ==查询进阶==
  - 排序查询
    - select ..   from 表名 order by 列 [ASC | DESC]
    - select ..   from 表名 order by 列 [ASC | DESC], 列  [ASC | DESC]
  - 聚合函数
    - 最大值 max(列)
    - 最小值 min(列)
    - 平均值 avg(列)
    - 求和 sum(列)
    - 个数 count(主键列 、*)
    - select 聚合函数(列)  from 表名
  - 分组查询
    - select 聚合函数(列)  from 表名  group by 列
    - select 聚合函数(列)  from 表名  group by 列 havinig 条件
    - having 、where 区别
  - 分页查询
    - select ...   from  表名 limit a, b
    - 页大小  pageSize, 页码 page
    - b =  pageSize
    - a  = pageSize *(page -1)
- 拆表
  - 原因：  修改数据更新数据维护数据不方便
- 外键
  - 作用： 保障数据的完整性， 维护表之间的关系
  - 特征：  类型要一致、引用的是主键列
  - 如何创建：  
  - 语法：
    - alter table 表名  add foreign key(列)  references 表(主键)

