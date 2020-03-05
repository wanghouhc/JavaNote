---
Typora-copy-images-to: img
---

# day18-连接池和事务

# 学习目标

- [ ] 能够理解连接池解决现状问题的原理
- [ ] 能够使用C3P0连接池
- [ ] 能够使用DRUID连接池
- [ ] 能够编写C3P0连接池工具类 
- [ ] 了解JDBC中的事务操作

# 第一章-自定义连接池 #

## 知识点-连接池概念

### 1.目标

+ 能够理解连接池解决现状问题的原理

### 2.讲解

#### 2.1为什么要使用连接池 ####

​	Connection对象在JDBC使用的时候就会去创建一个对象,使用结束以后就会将这个对象给销毁了(close).每次创建和销毁对象都是耗时操作.需要使用连接池对其进行优化.

​	程序初始化的时候，初始化多个连接,将多个连接放入到池(集合)中.每次获取的时候,都可以直接从连接池中进行获取.使用结束以后,将连接归还到池中.

#### 2.2.生活里面的连接池例子

- 老方式:

  ​	下了地铁需要骑车, 跑去生产一个, 然后骑完之后,直接把车销毁了.

- 连接池方式 摩拜单车:

  ​	骑之前, 有一个公司生产了很多的自行车, 下了地铁需要骑车, 直接扫码使用就好了, 然后骑完之后, 还回去

这样我们的单车可以复用，==连接池里面的数据库连接对象也可以循环使用（减少了频繁创建连接的代价）==


![img](img/tu_9.png)

#### 2.3连接池原理【重点】 ####

![img](img/tu_1.png)

1. 程序一开始就创建一定数量的连接，放在一个容器中（集合LinkedList），这个容器称为连接池(相当于碗柜/容器)。
2. 使用的时候直接从连接池中取一个已经创建好的连接对象。
3. 关闭的时候不是真正关闭连接，而是将连接对象再次放回到连接池中。

### 3.小结

- ![1567398430135](img\1571101710996.png)

## 自定义连接池-初级版本

### 1.目标

​	根据连接池的原理,  使用LinkedList实现连接池

### 2.分析

1. 编写一个类， 床架一个LinkedList对象去容纳我们的连接对象
2. 程序初始化的时候，创建5个连接对象，放到我们的池子
3. 获取连接的方法： 从池子获取，removeFirst()
4. 归还链接： 还到池子里去， addLast()

### 3.实现

```java
package com.itheima.a_pool;

import com.itheima.util.JdbcUtil;

import java.sql.Connection;
import java.util.LinkedList;


// 初级版本

/**
 * 1.初始化的时候，创建5个连接对象，放到池子里
 * 2.获取连接的方法,从池子获取
 * 3.使用完，归还连接,放到池子里去
 */
public class MyDataSource01 {


    // 初始化的时候，创建5个连接对象，放到池子里
    private static LinkedList<Connection> pool = new LinkedList<>();
    static{
        try {
            for(int i =1 ; i<=5 ; i++){
                Connection connection  = JdbcUtil.getConnection();
                    pool.add(connection);


            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    // 获取连接的方法
    public Connection getConnection(){
        // 从池子获取
        Connection connection = pool.removeFirst();
        return connection;
    }

    // 使用完，归还连接
    public void addBack(Connection connection){
        // 放到池子里去
        pool.addLast(connection);
    }
}

```

### 4.小结

- 程序初始化的时候，创建若干个(5)连接诶对象放到池子（LinkedList实现）
- 使用连接的时候，不再临时创建，而是从池子里获取 removeFirst()方法
- 使用完，连接对象不再直接销毁关闭了，而是归还到池子 addLast()方法

## 自定义连接池-进阶版本

### 1.目标

​	实现datasource完成自定义连接池

### 2.分析

​	在初级版本版本中, 我们定义的方法是getConnection(). 因为是自定义的.如果改用李四的自定义的连接池,李四定义的方法是getAbc(), 那么我们的源码就需要修改, 这样不方便维护. 所以sun公司定义了一个接口datasource,让自定义连接池有了规范

### 3.讲解

#### 3.1datasource接口概述

​	Java为数据库连接池提供了公共的接口：javax.sql.DataSource，各个厂商(用户)需要让自己的连接池实现这个接口。这样应用程序可以方便的切换不同厂商的连接池！

![img](img/tu_4.png)

#### 3.2代码实现

```java
package com.itheima.b_pool;

import com.itheima.util.JdbcUtil;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.LinkedList;
import java.util.logging.Logger;


// 进阶版本就是实现了  DataSource  接口， 就是规范了获取连接的方法名：  getConnection
public class MyDataSource02 implements DataSource {
    // 初始化的时候，创建5个连接对象，放到池子里
    private static LinkedList<Connection> pool = new LinkedList<>();
    static{
        try {
            for(int i =1 ; i<=5 ; i++){
                Connection connection = null;

                connection = JdbcUtil.getConnection();
                pool.add(connection);


            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public Connection getConnection() throws SQLException {
        // 从池子获取
        Connection connection = pool.removeFirst();
        return connection;
    }


    // 使用完，归还连接
    public void addBack(Connection connection){
        // 放到池子里去
        pool.addLast(connection);
    }


    public  static  int size(){
        return  pool.size();
    }
}

```

### 4.小结

- 进阶版本就是使用实现DataSource接口规范了getConnection的命名

















#### 4.1编写连接池遇到的问题

- 实现dataSource接口后,addBack()不能调用了.
- 能不能不引入新的api,直接调用之前的API.close(),但是这个close不是关闭,是归还

#### 4.2解决办法

- 继承

  + 条件:可以控制父类, 最起码知道父类的名字
  + ==不足：继承（要找到父类，父类在驱动里面，改变驱动没法搞了。）==


- 装饰者模式

  + 作用：改写已存在的类的某个方法或某些方法
  + 条件:
    + 增强类和被增强类实现的是同一个接口
    + 增强类里面要拿到被增强类的引用

## 自定义连接池-终极版本

### 1.目标

使用装饰者模式改写connection的close()方法, 让connection归还

### 2.讲解

#### 2.1装饰者模式

##### 2.1.1概述

- 什么是装饰者模式

  ​	装饰者模式，是 23种常用的面向对象软件的设计模式之一.  动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更加有弹性的替代方案。

  ​    装饰着模式是在不改变原有类文件和使用继承的情况下，通过创建一个包装对象动态地扩展一个对象的功能，相比生成子类更为灵活。

  ​	==装饰者的作用：改写已存在的类的某个方法或某些方法, 增强方法的逻辑==

- 使用装饰者模式需要满足的条件

  - 增强类和被增强类实现的是同一个接口
  - 增强类里面要拿到被增强类的引用
  
- 装饰模式实现的增强类：
  　　优点：内部可以通过多态技术对多个需要增强的类进行增强
    　　缺点：需要内部通过多态技术维护需要增强的类的实例。进而使得代码稍微复杂。

  

- 2.2.2装饰者模式的使用【重点】

实现步骤:

1. 增强类、被增强类实现同一个借口
2. 增强类拿到被增强类的引用
3. 需要增强的方法使用新的逻辑
4. 不需要增强额的方法使用原有逻辑

例子:

张三一个男的， 想追一个妹子， 张三就买了一辆车 Qq车， 能跑能停， 但是只能开60迈。

去改装店改装了一下，速度提升到100迈， 刹车没有动.....

```java
interface Car{
    void run();
    void stop();
}

class QqCar implements Car{
    ...
}

class WrapperCar implements Car{
    ...
}
```





+ 接口: Car.java

```java
/**
 * 车的规范
 */
public interface Car {

    void run();

    void stop();
}
```

+ 被增强的类: Qq.java

```java
/**
 * QQ厂商(被增强的类)
 */
public class Qq implements Car {


    @Override
    public void run() {
        System.out.println("Qq可以跑60迈...");
    }

    @Override
    public void stop() {
        System.out.println("Qq刹车...");

    }
}
```

+ 增强的类: WrapperCar.java

```java
/**
 * 改装店(增强的类)
 */
public class WrapperCar implements  Car{
    //需要把Qq车开进来(增强类(WrapperCar)里面需要得到被增强类(Qq)的引用)

   private Car car;
    public WrapperCar(Car car) {
        this.car = car;
    }

    @Override
    //需要增强(写自己的逻辑)
    public void run() {
        System.out.println("5s破百..");

    }

    @Override
    //不需要增强(调用之前的逻辑)
    public void stop() {
        car.stop();
    }
}
```

#### 2.2自定义连接池终极版本

##### 2.2.1分析

![](img\1573871304554.png)

##### 2.2.2实现

![1567672949309](img\1567672949309.png)

- MyConnection

  ```java
  
  public class MyConnection implements Connection {
  
      // 把车开进改装店
      private Connection connection;
      private LinkedList<Connection> pool = new LinkedList<>();
      public MyConnection(Connection connection, LinkedList<Connection> pool) {
          this.connection = connection;
          this.pool = pool;
      }
  
      // 需要被增强的逻辑，写新逻辑
      @Override
      public void close() throws SQLException {
          // 放到池子里去
          pool.addLast(connection);
      }
  
  
      // 不需要被增强的逻辑，使用原有逻辑
      @Override
      public PreparedStatement prepareStatement(String sql) throws SQLException {
          return connection.prepareStatement(sql);
      }
     //...其他方法
  }
  ```
  
- MyDataSource03

  ```java
  /**
   * 和 MyConnection 建立关系
   *  1. 初始化的时候存储的就是 MyConnection 对象
   *  2. 获取连接的时候，返回 MyConnection 对象（采用）
   */
  public class MyDataSource03  implements DataSource {
      // 初始化的时候，创建5个连接对象，放到池子里
      private static LinkedList<Connection> pool = new LinkedList<>();
      static{
          try {
              for(int i =1 ; i<=5 ; i++){
                  Connection connection = null;
  
                  connection = JdbcUtil.getConnection();
                  pool.add(connection);
  
  
              }
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
      @Override
      public Connection getConnection() throws SQLException {
          // 从池子获取
          Connection connection = pool.removeFirst();
  
          // 获取连接的时候包装一下，返回自定义的 myConnection 对象
          MyConnection myConnection = new MyConnection(connection, pool);
          return myConnection;
      }
  	// ... 其他方法
  }
  ```
  
  

### 3.小结

![](img\1572488828840.png)

# 第二章-第三方连接池

## 知识点-常用连接池

### 1.目标

+ 常用连接池

### 2.分析

​	通过前面的学习，我们已经能够使用所学的基础知识构建自定义的连接池了。其目的是锻炼大家的基本功，帮助大家更好的理解连接池的原理, 但现实是残酷的，我们所定义的 连接池 和第三方的连接池相比，还是显得渺小. 工作里面都会用第三方连接池. 

### 3.讲解

 常见的第三方连接池如下: 

+ ==C3P0==是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。C3P0是异步操作的，所以一些操作时间过长的JDBC通过其它的辅助线程完成。目前使用它的开源项目有Hibernate，Spring等。==C3P0有自动回收空闲连接功能==
+ 阿里巴巴-德鲁伊==Druid==连接池：Druid是阿里巴巴开源平台上的一个项目，整个项目由数据库连接池、插件框架和SQL解析器组成。该项目主要是为了扩展JDBC的一些限制，可以让程序员实现一些特殊的需求。
+ DBCP(DataBase Connection Pool)数据库连接池，是Apache上的一个Java连接池项目，也是Tomcat使用的连接池组件。dbcp没有自动回收空闲连接的功能。

## 知识点-C3P0 

### 1.目标

+ 掌握C3P0的使用

### 2.路径

1. c3p0介绍
2. c3p0的使用(硬编码)
3. c3p0的使用(配置文件)
4. 使用c3p0改写工具类

### 3.讲解

#### 3.1 c3p0介绍

![img](img/tu_3.jpg)

+ C3P0开源免费的连接池！目前使用它的开源项目有：Spring、Hibernate等。使用第三方工具需要导入jar包，c3p0使用时还需要添加配置文件c3p0-config.xml.
+ 使用C3P0需要添加c3p0-0.9.1.2.jar

#### 3.2c3p0的使用

##### 3.2.1通过硬编码来编写【了解】

步骤

+ 导包
+ 创建连接池对象
+ 获取连接
+ .....
+ 关闭资源；  connection.close() 


实现:

```java
ComboPooledDataSource cpds = new ComboPooledDataSource();
cpds.setDriverClass("com.mysql.jdbc.Driver");
cpds.setJdbcUrl("jdbc:mysql://localhost:3306/web17"); 
cpds.setUser("root");
cpds.setPassword("root");
Connection connection = cpds.getConnection();
```

##### 3.2.2 通过配置文件来编写【重点】

步骤:

+ 导包
+ 创建文件，  需要放在  ==src目录下， 并且命名一定是 c3p0-config.xml==
+ 创建连接池对象
+ 获取连接
+ 创建语句对象
+ 执行语句
+ 关闭所有资源

实现: 

+ 编写配置文件c3p0-config.xml，放在src目录下（注：==文件名一定不要改==)

```xml
<c3p0-config>
    <default-config>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/web17</property>
        <property name="user">root</property>
        <property name="password">root</property>
        <property name="initialPoolSize">5</property>
    </default-config>
</c3p0-config>
```

+ 编写Java代码 (会自动读取src目录下的c3p0-config.xml,所以不需要我们解析配置文件)

```java
DataSource ds = new ComboPooledDataSource(); 
```

#### ![1563760378588](img\1563760378588.png)

3.3使用c3p0改写工具类【重点】

​	我们之前写的工具类(JdbcUtils)每次都会创建一个新的连接, 使用完成之后, 都给销毁了; 所以现在我们要使用c3p0来改写工具类. 也就意味着,我们从此告别了JdbcUtils. 后面会使用c3p0写的工具类

```java
package com.itheima.util;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * 就是使用C3p0连接池子改造工具类
 */
public class C3p0Util {

    // 连接池一个就够了
    static ComboPooledDataSource dataSource = new ComboPooledDataSource();

    // 获取连接
    public static Connection getConnection() throws  Exception{
        // 获取连接诶
        return  dataSource.getConnection();

    }

    // 关闭所有资源
    public static void closeAll(ResultSet resultSet, Statement statement, Connection connection) throws  Exception{
        // 关闭资源
        if(resultSet != null ){
            resultSet.close();
        }
        if(statement != null ){
            statement.close();
        }
        if(connection != null ){
            // 已经是 归还连接了
            connection.close();
        }



    }
}

```

### 4.小结

- C3p0
  - 文件的使用步骤
    - 导包
    - 创建配置文件：  ==放在src目录下， 命名一定是 c3p0-config.xml==
    - 创建连接池对象
    - 从池子获取连接对象
    - 创建预编译语句对象
  - 执行语句对象
  - 关闭所有资源
  - 工具类的抽取C3p0Util
    - 就是把原来JdbcUtil里面创建连接改为---》 从池子里获取
    - close方法已经是归还连接







## 知识点-DRUID

### 1.目标

+ 掌握DRUID连接池的使用

### 2.路径

1. DRUID的介绍
2. DRUID的使用(硬编码方式)
3. DRUID的使用(配置文件方式)

### 3.讲解

#### 3.1DRUID介绍

​	Druid是阿里巴巴开发的号称为监控而生的数据库连接池，Druid是国内目前最好的数据库连接池。在功能、性能、扩展性方面，都超过其他数据库连接池。Druid已经在阿里巴巴部署了超过600个应用，经过一年多生产环境大规模部署的严苛考验。如：一年一度的双十一活动，每年春运的抢火车票。

Druid的下载地址：[https://github.com/alibaba/druid](https://github.com/alibaba/druid)  

DRUID连接池使用的jar包：druid-1.0.9.jar

![img](img/tu_7.png)



#### 3.2DRUID的使用

##### 3.2.1通过硬编码方式【了解】

步骤:

1. 引入jar包
2. 创建连接池对象
3. 获取连接
4. 创建语句对象
5. 执行、处理结果
6. 关闭资源

实现:

```java
//1. 创建DataSource
DruidDataSource dataSource = new DruidDataSource();
dataSource.setDriverClassName("com.mysql.jdbc.Driver");//设置驱动
dataSource.setUrl("jdbc:mysql://localhost:3306/web17");//设置数据库路径
dataSource.setUsername("root");//设置用户名
dataSource.setPassword("root");//设置密码

dataSource.setInitialSize(5);//设置初始化连接的数量


//2. 从数据源里面获得Connection
Connection connection = dataSource.getConnection();
```

##### 3.2.2 通过配置文件方式【重点】

步骤:

1. 导包
2. 创建文件， ==src目录下==， druid.properties文件
3. 读取文件，加载到properties对象里面
4. 创建连接池对象
5. 获取连接
6. 创建语句对象
7. 执行
8. 关闭所有资源

实现:

+ 创建druid.properties, 放在src目录下

```properties
url=jdbc:mysql://localhost:3306/web17
username=root
password=root
driverClassName=com.mysql.jdbc.Driver
```

+ 编写Java代码

```java
//0 根据druid.properties创建配置文件对象
Properties properties = new Properties();
// 关联druid.properties文件
InputStream is = DruidDemo.class.getClassLoader().getResourceAsStream("druid.properties");
properties.load(is);
//1. 使用数据源工厂创建DataSource
DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);

//2. 从数据源(连接池)获得连接对象
Connection connection = dataSource.getConnection();
```

#### 3.3作业(必做)

+ 把druid连接池也抽取一个工具类(参照C3P0Util)

### 4.小结

- druid的使用
  - 文件方式
    - 导包
    - 创建文件， src目录下  ， druid.properties
    - 读取文件，加载到properties对象
    - 创建连接池对象：  DruidDataSourceFactory.createDataSource(properties)
    - 获取连接
    - 创建语句对象
    - 执行语句对象
    - 关闭所有资源

# 第三章-JDBC事务的处理

## 知识点-JDBC事务介绍

### 1.目标

+ 掌握JDBC事务相关API

### 2.分析

之前我们是使用MySQL的命令来操作事务。接下来我们使用JDBC来操作事务. 先来学习下相关的API

### 3.讲解

| Connection中与事务有关的方法      | **说明**                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| setAutoCommit(boolean autoCommit) | 参数是true或false  如果设置为false，表示关闭自动提交，相当于开启事务; 类似sql里面的 start transaction; |
| void commit()                     | 提交事务; 类似sql里面的 commit;                              |
| void rollback()                   | 回滚事务; 类似sql里面的 rollback;                            |



## 案例-转账案例

+ 案例的准备工作

```sql
create table account(
    id int primary key auto_increment,
    name varchar(20),
    money double
);

insert into account values (null,'zs',1000);
insert into account values (null,'ls',1000);
insert into account values (null,'ww',1000);
```

### 1.需求

​	zs给ls转100, 使用事务进行控制

### 2.分析

#### 2.1 不使用事务的方式

1. 从C3p0Util里面获取连接
2. 创建语句对象2个（zs、ls）
3. 设置值
4. 执行语句 2个
5. 关闭资源

#### 2.2 使用事务的方式

1. 从C3p0Util获取连接
2. 开启事务  setAutoCommit(false)
3. 创建2个语句
4. 设置值
5. 执行语句2个
6. 一切都很OK， commit()
7. 异常情况，捕获之后回滚事务 rollback()
8. 关闭所有资源（finally里面）

### 3.实现

+ 代码实现

```java
package com.itheima.e_transform;

import com.itheima.util.C3p0Util;
import org.junit.Test;

import java.sql.Connection;
import java.sql.PreparedStatement;

public class TransformTransactionDemo {

    @Test
    public void test() throws Exception {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        PreparedStatement preparedStatement02 = null;
        try {
            connection = C3p0Util.getConnection();

            // 开启事务-- 关闭自动提交事务
            connection.setAutoCommit(false);

            // 张三---减少100块钱
            String sql01 = "update account set money = money - ? where name = ?";
            preparedStatement = connection.prepareStatement(sql01);
            preparedStatement.setDouble(1, 100);
            preparedStatement.setString(2, "zs");
            preparedStatement.executeUpdate();

            int i = 1/0;

            // 李四---增加100块钱
            String sql02 = "update account set money = money + ? where name = ?";
            preparedStatement02 = connection.prepareStatement(sql02);
            preparedStatement02.setDouble(1, 100);
            preparedStatement02.setString(2, "ls");
            preparedStatement02.executeUpdate();

            connection.commit();
        } catch (Exception e) {
            // 回滚事务
            connection.rollback();
            e.printStackTrace();
        } finally {
            preparedStatement.close();
            C3p0Util.closeAll(null, preparedStatement02, connection);
        }

    }
}

```

### 4.小结

- jdbc事务API
  - 开启事务： setAutoCommit(false)
  - 提交事务： commit()
  - 回滚：  rollback()







# 总结

### 重点内容

第三方连接池

- C3p0
  - 文件使用方式的步骤
    - 导包
    - 写一个配置文件 ，  ==放在src目录下， 命名为c3p0-config.xml==
    - 创建连接池对象
    - 获取连接‘
    - 创建预编译的语句对象
    - 设置值
    - 执行语句对象
    - 关闭所有资源
- Druid
  - 文件使用方式的步骤
    - 导包
    - 写文件， druid.properties， 放在src目录下
    - 读取文件，加载到 一个 properties对象里面
    - 创建连接池对象： DruidDataSourceFactory.createDataSource(properties)
    - 获取连接
    - 创建预编译语句对象
    - 设置值
    - 执行
    - 关闭所有资源

### 事务的API

- connection.setAutoCommit(false) : 手动开启事务
- connection.commit()
- connection.rollbakc()



如果有时间就看一下自定义。

