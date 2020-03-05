---
typora-copy-images-to: img
---

# day14-MySQL基础  #

# 学习目标

1. 能够理解数据库的概念
2. 能够安装MySQL数据库
3. 能够启动,关闭及登录MySQL
4. 能够使用SQL语句操作数据库
5. 能够使用SQL语句操作表结构
6. 能够使用SQL语句进行数据的添加修改和删除的操作
7. 能够使用SQL语句简单查询数据 

# 第一章-数据库概述

## 知识点-数据库的介绍 

### 1.目标

- 知道什么是数据库
- 能够说出和Java相关的常用数据库 

### 2.分析

目前来说如果我们要进行数据存储，有几种方式:

- 内存：断电数据丢失
- 普通文本文件保存数据：开发困难、数据存储形式组织太混乱、增删改查操作效率低下

### 3.讲解

#### 3.1什么是数据库   

​	数据库（DataBase，DB）：指长期保存在计算机的存储设备==(硬盘==)上，按照==一定规则组织==起来，可以被各种用户或应用共享的可以快速查询的数据集合. 还是以文件的方式存在服务器的电脑上的。

​	说白了: 数据库是数据仓库, 用来保存数据的

#### 3.2常见的关系型数据库

- MYSQL	：开源免费的数据库，中小型的数据库，已经被Oracle收购了。MySQL6.x版本也开始收费。后来Sun公司收购了MySQL，而Sun公司又被Oracle收购
- Oracle：收费的大型数据库.Oracle公司的产品.Oracle收购SUN公司，收购MYSQL.
- DB2：IBM公司的数据库产品,收费的.银行系统中.
- SQLServer：MS公司.收费的中型的数据库.
- SyBase：已经淡出历史舞台.提供了一个非常专业数据建模的工具 PowerDesigner.
- SQLite: 嵌入式的小型数据库,应用在手机端.

![img](img/tu_1.jpg)

### 4.小结

- 数据库的概念
  - 保存数据的仓库database 、DB
  - 持久化保存，利于增删改查； 按照一定规则组织存储的。
- mysql



## 知识点-数据库结构【重点】

### 1.目标

+ 掌握数据库结构

### 2.分析

​	数据库是用来存储数据的,那么到底通过什么样的方式来存的. 结构是怎么样的呢?

### 3.讲解

#### 3.1 数据库结构

​	数据库结构依赖于数据库管理系统(Database Manage System：DBMS)，通过它可以管理我们的多个数据库（Database）。

​	数据库本身不能存储数据，数据真实存储在表（table）里面。

​	表是以行、列组织存储数据的，一行叫一个行记录，一列叫做字段(column)或者属性。

​	数据库管理系统、数据库和表的关系如图所示：

![img](img/tu_16.png)



### 4.小结

- 数据库结构
  - 依赖于一个软件： DBMS，数据库管理系统
  - 通过DBMS可以管理多个数据库
  - 数据存放在表table里面，是以行（行记录row）、列（字段、属性column）存储的。

# 第二章-数据库的卸载,安装,启动,登录   #

## 实操-MySql的卸载、安装

### 1.目标

+ 掌握MySQL的卸载和安装

### 2.路径

​	我们要使用MySql数据库,就需要先安装. 如果第一次安装失败了,就需要卸载,再安装.

1. mysql的安装
2. mysql的卸载  

### 3.讲解

#### 3.2mysql的卸载

具体参考文档: `资料\01_MySQL安装与卸载\MySQL卸载手册.doc`

![1555490988722](img/1555490988722.png) 

#### 3.2mysql的安装(掌握)

具体参考文档: `资料\01_MySQL安装与卸载\MySQL安装图解.docx`

![1555490978511](img/1555490978511.png) 

#### 安装重要事项

- ==字符集设置（utf8）==
- ==安装目录==
- ==密码设置(不要忘记密码了)==
- ==环境变量设置==

### 4.小结

+ 安装
  + 参考手册执行就可以了
  + 比如你安装了2次，30分钟； 找同学、老师
  + 注意事项： 字符集编码 utf8
+ 卸载（了解）



## 实操-数据库服务的启动和登录,退出

### 1.目标

​	我们刚刚把数据库安装成功了,下面就需要进行数据库的启动,登录和退出

### 2.步骤

+ 数据库服务的启动
+ Mysql的登录(命令行和客户端方式)
+ MySql的退出

### 3.讲解

#### 3.1数据库服务的启动

##### 3.1.1 方式一通过界面启动

![](img\1565595822797.png)

开始菜单--》计算机管理--》服务与应用：

![img](img/tu_10.png)

##### 3.1.2方式二通过DOS命令方式启动

![img](img/tu_11.png)



![img](img/tu_12.png)





MySQL是一个需要账户名密码登录的数据库，登陆后使用，它提供了一个默认的root账号，使用安装时设置的密码即可登录.（root）

#### 3.2登录

##### 3.2.1命令行

- 格式一

```mysql
mysql -u 用户名 -p       #提示输入密码   # 默认是连接本地  127.0.0.1 localhost
mysql -u 用户名 -p密码   #提示输入密码
```

- 格式二  

```mysql
mysql -h ip地址 -u 用户名 -p密码    注: 127.0.0.1 代表本机的IP地址   
-h：表示指定MySQL服务器所在的机器ip或者主机域名
```

##### 3.2.2图形化工具

- 安装

![1570504577089](img\1570504577089.png)

- 连接

  ![1555917461553](img\1570504607801.png) 

  ![1555917516520](img\1570504646491.png) 

#### 2.3退出

##### 2.3.1命令行

```
输入: quit或exit
```

##### 2.3.2图形化工具

- 右键关闭

### 4.小结

- 数据库登录语法

  ```mysql
  mysql -u 用户名 -p
  mysql -u 用户名 -p密码
  mysql -h 主机ip地址 -u 用户名 -p密码
  ```
  


# 第三章-sql概述 #

## 知识点-Sql介绍

### 1.目标

+ 掌握什么是SQL

### 2.步骤

+ 介绍Sql的概念
+ sql的语法的注意事项
+ 介绍sql的类别

### 3.讲解

#### 3.1.什么是sql?

- SQL：Structure Query Language。（结构化查询语言）,操作数据库的手段，通过sql操作数据库(操作数据库,操作表,操作数据)
- SQL被美国国家标准学会（ANSI）确定为关系型数据库语言的美国标准，后来被国际化标准组织（ISO）采纳为关系数据库语言的国际标准
- 各数据库厂商(mysql,oracle,sql server)都支持ISO的SQL标准。
- 各数据库厂商在标准的基础上做了自己的扩展。   各个数据库自己特定的语法 --方言


![img](img/tu_17.png)

#### 3.2sql的规则

+ 每条语句以分号;(英文输入法的分号，半角符号)结尾(命令行里面需要)，如果在navicat,sqlyog,java代码中不是必须加的。 
+ SQL在window中不区分大小写，关键字中认为大写和小写是一样的 

#### 3.3sql的分类

- DDL：数据定义语言 (Data Definition Language) 如：操作数据库，操作表
- DML：数据操纵语言(Data Manipulation Language)，如：对表中的记录操作增删改
- DQL ：数据查询语言(Data Query Language)，如：对表中的查询操作,主要用来做查询使用
- DCL ：数据控制语言(Data Control Language)，如：对用户权限的设置，很少用，主要DBA或者运维人员才使用  

### 4.小结

- SQL
  - 是什么？ 操作数据库的语言、手段、方式
  - 语法限制： dos命令行，语句都是以;结尾的。
- 分类
  - DDL语言： 操作数据库、表
  - DML语言： 对数据进行增删改
  - DQL语言： 对数据进行查询
  - DCL语言：权限进行控制

# 第四章-DDL对数据库的CRUD #

## 知识点-对数据库的CRUD

### 1.目标

​	我们把Sql介绍完成了, 那下面就通过Sql语句对数据库进行CRUD（create、read、update、delete）

### 2.步骤

+ 创建数据库(掌握)
+ 查看数据库
+ 选择使用某个数据库(掌握)
+ 修改数据库
+ 删除数据库(掌握)

### 3.讲解

#### 3.1创建数据库

- 语法

```mysql
create database 数据库名 [character set 字符集][collate  校对规则]     注: []意思是可选的意思
```

字符集(charset)：是一套符号和编码。mysql里面统一编码是叫做：==utf8==，不是utf-8。

- 练习

创建一个web14_1的数据库（默认字符集)  

```mysql
create database web14_1;
```

创建一个web14_2的数据库,指定字符集为gbk(了解) 

```mysql
create database web14_2 character set gbk;
```



#### 3.2查看所有的数据库

##### 3.2.1查看所有的数据库

- 语法  

```mysql
show databases; 
```

##### 3.2.2查看数据库的定义结构【了解】

- 语法

```mysql
show create database 数据库名;
```

- 查看web14_1这个数据库的定义

```mysql
show create database web14_1; 
```

#### 3.3 选择使用某个数据库(掌握)

- 切换数据库, 选定哪一个数据库

```mysql
use 数据库名;   		//注意: 在创建表之前一定要指定数据库. use 数据库名
```

- 练习: 使用web14_1 

```mysql
use web14_1;
```

- 查看正在使用的数据库

```mysql
select database();
```



#### 3.4修改数据库【了解】

- 语法 

```mysql
alter database 数据库名 character set 字符集;
```

- 修改web14_1这个数据库的字符集(gbk)

```mysql
alter database web14_1 character set gbk;
```

**注意：**

- ==是utf8，不是utf-8==
- 不是修改数据库名

#### 3.5删除数据库

- 语法  

```mysql
drop database 数据库名;
```

- 删除web14_2数据库

```mysql
drop database web14_2;
```

#### 3.6 练习

- 创建一个web14_3的数据库，字符集指定为gbk 
- 查看它的定义结构
- 修改web14_3的字符集为utf8(==实际开发中，设计时决定好，一般不需要变更==)
- 选择使用它
- 查看当前使用的是不是web14_3
- 删除web14_3数据库

### 4.小结

- DDL语言操作数据库的CRUD
  - 创建数据库【掌握】： `create database 数据库名`
  - 查看数据库的定义结构： `show create database 数据库名`
  - 查看所有的数据库： `show databases;`
  - 使用某个数据库【掌握】： `use 数据库名`
  - 查看当前正在使用的数据库： `select database()`
  - 删除数据库【掌握】： `drop database 数据库名`

# 第五章-DDL对表的CRUD

## 知识点-创建表【重点】

### 1.目标

​	我们第四章已经把数据库的CRUD讲解完了,下面我们就学习创建表

### 2.步骤

+ 创建表的语法介绍
+ mysql常见的数据类型
+ mysql约束

### 3.讲解

#### 3.1语法

```mysql
create table 表名(
	列名(字段) 类型 [约束]， 
	列名 类型 [约束]，
	列名 类型 [约束]
);
```

#### 3.2 类型

![img](img/tu_4.png)

1. 数值类型
   1. 整型： 最常用的是int---》java的int(Integer)、bigint---》java的long、tinyint---》java的byte
   2. 浮点型： 定义时指定参数(M,D)，分别表示 总长度位数、小数位数。salary double(4,2) -->标识的范围就是-99.99~99.99。
2. 文本类型
   1. char(size)：固定长度字符，char(20)-->存储'zs'-->占用空间还是20
   2. varchar(size)：可变字符，varchar(20)-->存储'zs'-->占用空间2
   3. 实际中，基本上采用varchar；身份证、手机号特定长度采用char.
   4. ==字符串需要使用 单引号( ' )或双引号" 进行引用起来。==
3. 二进制类型
   1. 经验法则：一个高清视频4G，直接存储在数据里面吗？存储一个资源文件url；实际的资源-->专有的文件服务器。
4. 时间日期类型
   1. date：日期，格式 2019-01-01
   2. datetime：日期时间，格式 2019-01-01 12:30:00；32位的机器仅仅表示到2038年。
   3. timestamp：时间戳，一般可用于记录 自动插入时间、修改时间

#### 3.3 约束

- 即规则,规矩 限制;  
- 作用：保证用户插入的数据保存到数据库中是符合规范的

![img](img/tu_18.png)

**约束种类:** 

- ==not null==: 非空 ;  eg: name varchar(40) not null, 当前的name 列里面不可以有null数据
- ==unique==:唯一约束, 后面的数据不能和前面重复;  eg: `cardNo varchar(20) unique `


- ==primary key==；主键约束(非空+唯一);    一般用在表的id列上面.  一张表基本上都有id列的, id列作为唯一标识的 
- ==auto_increment==: ==**自动增长,必须是设置了primary key之后,才可以使用auto_increment**==
- id int  primary key   **auto_increment**;  id不需要我们自己维护了, 插入数据的时候直接插入null, 自动的增长进行填充进去, 避免重复了.  

id列:

1. 给id设置为int类型, 添加主键约束,  自动增长
2. 或者给id设置为字符串类型,添加主键约束,  不能设置自动增长

#### 3.4练习

- 练习一：创建一张学生表(含有id字段,姓名字段,性别字段. id为主键自动增长)

```mysql
create table student(
	id int primary key auto_increment,
	name varchar(40),
	sex int
);
```

- 练习二：创建一张员工表（id 主键自增长，姓名 要求不能为空，手机号唯一，性别，收入，入职时间精确到秒）

```mysql
create table employee(
	id int primary key auto_increment,
    name varchar(40) not null,
    phone char(11) unique,
    sex char(1),
    salary double,
    employedTime datetime
);
```



### 4.小结

- 创建表语法

  ```mysql
create table 表名(
  	字段 类型 [约束],
      字段 类型 [约束]
  );
  ```
  
- 数据类型

  - 数值
    - 整型： int、bigint 、tinyint
    - 浮点类型： float、double；  double(4, 2)： 表示  -99.99----99.99
  - 文本、二进制
    - char(size)： 定长字符串；   char(20)--->存储了'zs'---占有空间：20
    - varchar(size)： 可变长度 ；  varchar(20)-->存储'zs'---占有空间：2
  - 日期时间
    - 日期： date：  2019-11-11
    - 日期： datetime
    - 时间戳： timestamp

## 知识点-查看表【了解】

### 1.目标

我们把表创建好了, 下面就来介绍查看表

### 2.步骤

+ 查看当前数据库所有的表
+ 查看表的定义结构

### 3.讲解

#### 3.1查看所有的表

```mysql
show tables;
```
#### 3.2查看表的定义结构

+ 语法

  desc  表名；

+ 练习: 查看student表的定义结构

```mysql
desc student;  --  desc是 descibe 的简写
DESCRIBE student;
```



### 4. 小结

1. 查看数据库有多少表
   1. show tables;
2. 查看表的定义结构
   1. ```desc 表名```

## 知识点-修改表【掌握】

### 1.目标

​	我们表创建好了, 如果要增加一列,要删除一列呢? 那下面就来讲解修改表

### 2.步骤

+ 增加列
+ 修改列的类型约束
+ 修改列的名称，类型，约束
+ 删除列
+ 修改表名

### 3.讲解

#### 3.1语法

对表的修改操作 一般使用 ```alter table  表名```

- 增加==(```add```)==一列： ```alter table 表名 add 字段 数据类型 约束```

  - 给学生表增加一个grade字段

  - `alter table student add grade varchar(20) not null;`

- 列类型变更==（```modify```）==：```alter table 表名 modify 字段 类型 约束```

  - 给学生表的sex字段改成字符串类型

  - `alter table student modify sex varchar(10);`

- 变更列的名字==(```change```)==: ```alter table 表名 change 旧列 新列 类型 约束```

  -  给学生表的grade字段变更为class字段

  - `alter table student change grade class varchar(20);`

- 删除一列==(```drop```)==：```alter table 表名 drop 列名```

  - 把class字段删除

  - `alter table student drop class;`

- 修改表名==(```rename```)==: ```rename table 旧表名 to 新表名```

  - 把学生表修改成老师表(了解)
  
  - `rename table student to teacher`
    
    


#### 3.2 练习

  - 创建一张用户表user（有id要求自增长，用户名唯一，密码固定长度8位）
  - 给用户表增加一列性别sex，使用整型


  - 将性别列sex类型改为字符型
  - 将sex列改为gender
  - 删除gender列
  - 重命名用户表改为账户表account

### 4.小结

- DDL修改表
  - 增加一列： `alter table 表名 add 字段 类型 [约束]`
  - 修改列的类型： `alter table 表名 modify 字段  类型 [约束]`
  - 变更列的名字： `alter table 表名 change 旧列 新列 字段 [约束]`
  - 删除列： `alter table 表名 drop 列名`
  - 重命名表： `rename table 旧名 to 新表名`

## 知识点-删除表【掌握】

### 1.目标

​	表创建好了, 我们还可以删除。 掌握表的删除

### 2.步骤

+ 删除表

### 3.讲解

+ 语法

  ```drop table 表名；```

+ 把teacher表删除

```mysql
drop table teacher;
```

### 4.小结

1. drop table teacher; 

   

# 第六章-DML和DQL对表里面数据的CRUD【重点】 #

+ 准备工作: 创建一张商品表(商品id,商品名称,商品价格,商品数量.) 


```mysql
create table product(
	pid int primary key auto_increment,  //只有设置了auto_increment id列才可以赋值为null
	pname varchar(40),
	price double,
	num int
);
```

## 知识点-插入记录【重点】

### 1.目标

+ 掌握插入记录

### 2.步骤

+ 两种方式插入数据
+ 使用命令行操作时候乱码的解决

### 3.讲解

+ 方式一:  插入指定列, 如果没有把这个列进行列出来, 以null进行自动赋值了.


```mysql
insert into 表(列，列..) values(值，值..)；
```
```mysql
insert into product(id,name) values(null,'Mac'); 
```

现在price和num没有列出来, 那么price和num就是null。

> 注意:  如果设置了非空约束，但是插入数据时没有插入该列的值, 会报错，除非该列设置了主键自增长。

+ 方式二: 插入所有的列


```mysql
insert into 表 values(值，值....);           

eg:
insert into product values(null,'苹果电脑',18000.0,10);
insert into product values(null,'华为5G手机',30000,20);
insert into product values(null,'小米手机',1800,30);
insert into product values(null,'iPhonex',8000,10);
insert into product values(null,'苹果电脑',8000,100);
insert into product values(null,'iPhone7',6000,200);
insert into product values(null,'iPhone6s',4000,1000);
insert into product values(null,'iPhone6',3500,100);
insert into product values(null,'iPhone5s',3000,100);

insert into product values(null,'方便面',4.5,1000);
insert into product values(null,'咖啡',11,200); 
insert into product values(null,'矿泉水',3,500);
```

**命令行插入中文数据报错:**

![img](img/tu_3.png)

- 关闭服务,  net stop mysql
- 在数据库软件的安装目录下面, 修改配置文件 my.ini中客户端的编码为gbk


![1535858746067](img/1535858746067.png)

- 重新打开命令行,开启服务, ```net start mysql```

### 4.小结

- DML写数据

  - 语法 

    ```mysql
    -- 指定列
    insert into 表名(列, 列...)  values(值, 值...)
    -- 所有列
insert into 表名 values(值,值...)
    ```
    
    

## 知识点-更新记录

### 1.目标

我们数据插入成功了， 还可以对已有的数据进行更新。

### 2.步骤

+ 更新数据

### 3.讲解

#### 3.1语法

```mysql
update 表名 set 列 = 值 ,列= 值 [where 条件]
```
#### 3.2练习

- 将所有商品的价格修改为5000元

```mysql
update product set price = 5000;
```

- 将商品名是苹果电脑的价格修改为18000元

```mysql
UPDATE product set price = 18000 WHERE name = '苹果电脑';
```

- 将商品名是苹果电脑的价格修改为17000,数量修改为5

```mysql
UPDATE product set price = 17000,num = 5 WHERE name = '苹果电脑';
```

- 将商品名是方便面的商品的价格在原有基础上增加2元

```mysql
UPDATE product set price = price+2 WHERE name = '方便面';
```

### 4.小结

1. 对数据库修改操作

   1. 语法： 
   
      ```mysql
   update 表名 set 列 = 值, 列=值  [where 条件]
      ```
   
      
   
   

## 知识点-删除记录

### 1.目标

+ 掌握记录的删除

### 2.步骤

+ 使用delete删除
+ 使用truncate删除
+ delete删除和truncate删除的区别

### 3.讲解

#### 3.1delete

+ 语法

```mysql
delete from 表 [where条件];     注意: 删除数据用delete，delete操作一般加上where条件
```

- 练习

  1. 删除表中名称为’苹果电脑’的记录

  2. 删除价格小于3000的商品记录

  3. 删除表中的所有记录

- 参考答案

删除表中名称为’苹果电脑’的记录

```mysql
delete from product where pname = '苹果电脑';
```

删除价格小于5001的商品记录

```mysql
delete from product where price < 3000;
```

删除表中的所有记录

```mysql
delete from product;
```

#### 3.2truncate

```mysql
truncate table 表名;
```

![1565233419791](img\1565233419791.png)

delete  和truncate区别【面试题】  

- DELETE 删除表中的数据，表结构还在;  删除后的数据可以找回,一条一条的删除. 
- TRUNCATE 删除是把表直接DROP掉，然后再创建一个同样的新表。删除的数据不能找回。执行速度比DELETE快。

### 4.小结

1. 删除操作： `delete from 表名 where 条件`

2. 经验法则

   1. 实际中，很少真正删除数据的--物理删除。通常采用逻辑删除。
   2. 物理删除： 在数据库中真正把数据清除掉。
   3. 逻辑删除： 采用某个字段（状态），记录数据的删除、有效的状态。

   | id   | name | status |
   | ---- | ---- | ------ |
   | 1    | 张三 | 1      |
   | 2    | 李四 | 0      |

   

## 知识点-查询记录【重点】

### 1.目标

​	我们上面讲解了对数据的增删改, 下面就来重点讲解数据的简单查询.  

### 2.步骤

+ 基本查询语法
+ 查询所有的列
+ 查询某张表特定列
+ 去重查询
+ 别名查询
+ 运算查询(+,-,*,/等)
+ 基本条件查询

### 3.讲解

#### 3.1基本查询语法

```mysql
select  [*] [列名 ,列名] [列名 as 别名 ...] [distinct 字段] from 表名
select  [*] [列名 ,列名] [列名 as 别名,列名 as 别名 ...] [distinct 字段] from 表名
-- select [结果集] from 表名 [where 条件]
-- 标点符号都是英文半角符
-- [] 表示可选项
```
#### 3.2简单查询

##### 3.2.1 查询所有的列的记录

+ 语法

```mysql
select * from 表
```

+ 查询商品表里面的所有的列

```mysql
select * from product;
```

##### 3.2.2查询某张表特定列的记录

+ 语法

```mysql
select 列名,列名,列名... from 表
```

+ 查询商品名字和价格

```mysql
select pname, price from product;
```

##### 3.2.3 去重查询 

+ 语法

```mysql
SELECT DISTINCT 字段名 FROM 表名;   //要数据一模一样才能去重
```

+ 去重查询商品的价格

```mysql
select distinct price from product;
```

> 注意点: 去重针对某列, distinct前面不能先出现列名

##### 3.2.4 别名查询    

+ 语法

```mysql
-- 给字段取别名
select 列名 as 别名 ,列名 别名 from 表   //列别名  as可以省略不写
-- 给表取别名
select 别名.* from 表 as 别名      //表别名(给表取别名常用语多表查询, 后2天会具体讲)
```


- 查询商品名称和商品价格，商品价格通过别名‘价格’来显示


```mysql
select pname , price as 价格 from product;
```

##### 3.2.5运算查询(+,-,*,/等)

- 把商品名，和商品价格+10查询出来


```mysql
select pname ,price+10 from product;
```

>  注意

- 运算查询字段,字段之间是可以的  
- 字符串等类型可以做运算查询，但结果没有意义


#### 3.3条件查询

##### 3.3.1语法

```mysql
SELECT xx FROM 表名 WHERE 条件;  //取出表中的每条数据，满足条件的记录就返回，不满足条件的记录不返回
-- 其实就是在基本查询后面增加了 where 条件
```

![img](img/tu_2.png)

1. ==between...and..==    闭区间(包含临界值)  
+ eg: price between 3000 and 6000 等价于  3000<=price<=6000
  
   + 查询价格在[3000,6000]的产品名称、价格
   
   + ```mysql
     SELECT pname, price FROM product WHERE price BETWEEN 3000 AND 6000;
     ```
   
     
   
2.  ==in 查询多个==

```mysql
select * from product where id  = 1;
select * from product where id  = 3;
select * from product where id  = 5;
select * from product where id  = 7;
select * from product where id in(1,3,5,7);
# 查询id为1、3、6、9的所有产品信息
select * from product  where id in(1,3,6,9);
```

3. ==like== 模糊查询  一般和_或者%占位符配合使用

   + _ 占一位（有且仅有一位）
   + % 占0~n位

   应用场景, 查询姓张的用户,  name like '张_'  查询姓张的 并且名字有两个字的

   ​											   name like '张%'  查询姓张的

   ```mysql
   ## 查询以 iPhone 开头的产品名称的所有产品信息
   SELECT * FROM product  WHERE pname  LIKE 'iPhone%';
   
   # 查询 iPhone 开头的产品名称，且后面只有一位字符的产品信息
   SELECT * FROM product WHERE pname LIKE 'iPhone_';
   ```

   

4. 并且   where  条件 ==and== 条件 and 条件....

   ```mysql
   
   # 查询 iPhone 开头的产品名称并且价格在2000以下。
   SELECT * FROM product  WHERE pname LIKE 'iPhone%' AND price < 5000;
   ```

   

5. 或者任意  where 条件 ==or== 条件 or 条件

   ```mysql
   # 查询价格小于1000的和价格大于10000的商品信息
   SELECT * FROM product  WHERE price  < 1000 OR price > 10000;
   ```

6. 非，相反；  ==not== 条件

   ```mysql
   # 查询金额不大于1000的产品
   select * from product where not(price > 1000)
   ```

   

##### 3.3.2练习


- 查询商品价格>3000的商品


- 查询id=1的商品

- 查询id<>1的商品

- 查询价格在3000到6000之间的商品

- 查询id为1，5，7，15的商品

- 查询商品名以iPho开头的商品(iPhone系列) 

- 查询商品价格大于3000并且数量大于20的商品   (条件 and 条件 and...)


- 查询id=1或者价格小于3000的商品 



### 4.小结

- DQL

  - 语法

    ```mysql
    select  结果集  from 表名 ;
    select  *  from 表名 ;
    select  列, 列  from 表名 ;
    select  列 as 别名,列 as 别名  from 表名 ;
    select  distinct 列  from 表名 ;
    
    -- 条件查询
    select  结果集  from 表名 where 条件;
    ```

  - between ... and ... : 闭区间

  - in(set)： in(1,2,3)

  - 模糊查询： like  '模式匹配'

    - 占位符  _  ： 有且仅有一位字符
    - %  ：  任意字符

  - and 、or、not

# 总结

- 数据库(database)介绍

  - 数据库是什么？ 存储数据的仓库
  - 常见的数据库： mysql
  
- 数据库结构

  - 依赖于一个软件 数据库管理系统DBMS
  - DBMS可以管理多个数据库
  - 数据库可以放多张表
  - 表是以行（行记录row）、列（字段、属性、column）

- SQL语言

  - sql语言： 操作数据库的一种语言、手段、方式
  - 语法规则： 一行语句以;结束
  - 分类
    - DDL语言: 操作数据库、表
    - DML： 对数据进行增删改操作
    - DQL： 查询
    - DCL： 权限操作

- DDL语言对数据库的CRUD

  - ==创建数据库==：  `create database 数据库名`
  - ==使用某个数据库==：  `use 数据库名`
  - ==删除数据库==： `drop database 数据库名`

- DDL操作表的CRUD

  - 创建表

    ```mysql
    create table 表名(
    	字段 类型 [约束],
        字段 类型 [约束]
    );
    ```

    - 数据类型
      - 数值
        - 整型：  int 、bigint、tinyint
        - float、double :  ==float(4, 2)表示范围 -99.99----99.99==
      - 文本二进制
        - char(size): 定长字符：  ==char(20)--->'zs'-->实际占有长度也是20==
        - varchchar(size): 可变长度字符：  ==varchchar(20)--->'zs'-->实际占有长度就是实际存储的长度2==
      - 时间日期
        - date ：  2019-11-11
        - datetime:  2019-11-11 12:29:00
        - timestamp:到时分秒

  - ==对表的修改==

    - 增加列： `alter table 表名 add 字段 类型 [约束]`
    - 删除列：  `alter table 表名 drop 列`
    - 对列类型变更：  `aleter table 表名 modify 列 类型 [约束]`
    - 变更列名：  `alter table 表名 change 旧列 新列 类型 [约束]`
    - 重命名表：  `rename table 旧表名 to 新表名`

  - 查看表

    - 查看数据库所有的表： `show tables`
    - 查看结构：  `desc 表名`

  - ==删除表==

    - drop table 表名

- ==DML操作==

  - 插入：  
    - insert into 表名(列, 列...)  values(值,值...)
    - insert into 表名 values(值,值...)
  - 修改
    - update 表名 set 列 =  值 ,  列=值  [where 条件]
  - 删除
    - delete from 表名  [where 条件]

- ==DQL==

  ```mysql
  select 结果集  from  表名 ;
  select 列, 列...  from  表名 ;
  select 列 as  别名  from  表名 ;
  select distinct 列  from  表名 ;
  
  select 结果集  from  表名 where 条件;
  ```

  - 不等于：  <>
  - 闭区间：  between .. and ...
  - 散列集合：  in(值, 值)
  - 模糊查询： _ 有且仅有一位；  % 任意位数
  - and 、or 、not 

# 补充资料(了解)



## 选择合适的数值数据类型

- 如果整型数据范围在1-255之间，选择 ```TINYINT UNSIGNED``` 是最好的--占用空间小
- 对于基本的四则运算（+、-、*、/），所以如果不是要求非常精确的场景下，选择```DOUBLE``` 已经足够了。
- 对于更高的精度，你可以将其使用 ```BIGINT``` 类型来存储，对于64位的整数计算已经OK了，后面你可以在需要的场景下转换为浮点型表示即可(比如在Java程序中使用`BigDecimal`类型进行精确的计算，得到精确的结果后转换为整型保存到数据库)。
- 对于表示性别、婚否此类的字段可以采用 ```TINYINT```表示。
- 对于证件、手机号等可以采用```char```类型存储。

## MySQL整型数据类型、占用字节空间、表示范围概览

![1562838414794](img\1562838414794.png)





