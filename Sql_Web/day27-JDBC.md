---
typora-copy-images-to: img
---

# day17-JDBC

## 学习目标

1. 能够理解JDBC的概念
2. 能够使用DriverManager类
3. 能够使用Connection接口
4. 能够使用Statement接口
5. 能够使用ResultSet接口
6. 能够说出SQL注入原因和解决方案
7. 能够通过PreparedStatement完成增、删、改、查
8. 能够完成PreparedStatement改造登录案例

## 一,JDBC概述 ##

### 1.为什么要使用JDBC ###

1. 没有JDBC

![img](img/tu_1.jpg)

2. 有了JDBC后

![img](img/tu_2.png)


### 2.JDBC的概念 ###

#### 2.1什么是JDBC

​	JDBC:java database connectivity: sun公司为了简化和统一java连接数据库,定义的一套规范(API,接口). 

#### 2.2JDBC和数据库驱动的关系

​	接口(JDBC)与实现(驱动jar包)的关系


## 二,开发第一个JDBC程序 ##

### 1.开发第一个JDBC程序

#### 1.1准备工作:

+ 数据库的准备

  ```sql
  create database web17;

  use web17;

  create table user(
  	id int primary key auto_increment,
  	username varchar(20),
  	password varchar(20),
  	nickname varchar(20)

  );

  INSERT INTO `USER` VALUES(null,'zs','123456','老张');
  INSERT INTO `USER` VALUES(null,'ls','123456','老李');
  INSERT INTO `USER` VALUES(null,'wangwu','123','东方不败');
  ```

+ 驱动jar包导入项目

  ![img](img/tu_11.png)



![1554454931523](img/1554454931523.png)

 

 ![1554454953928](img/1554454953928.png)

#### 1.2开发步骤

+ 注册驱动(先导入jar包)
+ 获得连接
+ 创建执行SQL语句的对象
+ 执行SQL语句,处理结果
+ 释放资源



#### 1.3代码实现

```java
package com.itheima.a_jdbc;

import org.junit.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * jdbc快速入门
 */
public class JdbcDemo {

    public static void main(String[] args) throws Exception {
        //1.注册驱动
        //DriverManager.registerDriver(new Driver());
        //类全限定名(带包名), 加载Driver类, 静态代码块就会执行, 驱动就注册了
        Class.forName("com.mysql.jdbc.Driver");

        //2.获得连接(连接数据库)
        //连接数据库路径
        String url="jdbc:mysql://localhost:3306/web17";
        String user = "root";
        String password= "123456";
        Connection connection = DriverManager.getConnection(url, user, password);

        //3.创建执行sql语句的对象
        Statement statement = connection.createStatement();

        //4.执行sql语句, 处理结果
        String sql = "select * from user";
        ResultSet resultSet = statement.executeQuery(sql);

        while (resultSet.next()){
            System.out.println(resultSet.getInt("id"));
            System.out.println(resultSet.getString("username"));
            System.out.println(resultSet.getString("password"));
            System.out.println(resultSet.getString("nickname"));
            System.out.println("**********");
        }


        //5.释放资源(先创建的后关闭)
        if(resultSet != null){
            resultSet.close();
        }

        if(statement != null){
            statement.close();
        }
        if(connection != null){
            connection.close();
        }

    }
}
```




## 三,JDBC API详解 ##

![img](img/tu_3.png)

### 1.java.sql.Drivermanager###

1.registerDriver(Driver driver) ;注册驱动

	static {
		try {
			java.sql.DriverManager.registerDriver(new Driver());
		} catch (SQLException E) {
			throw new RuntimeException("Can't register driver!");
		}
	}

 翻阅源码发现,通过API的方式注册驱动,Driver会new两次,所有推荐这种写法:

	Class.forName("com.mysql.jdbc.Driver");
2. getConnection(String url, String user, String password) ;与数据库建立连接![img](img/tu_8.png)


### 2.java.sql.Connection接口 ###

​	接口的实现在数据库驱动中。所有与数据库交互都是基于连接对象的。

1. createStatement() ;创建执行sql语句对象
2. prepareStatement(String sql) ;创建预编译执行sql语句的对象


### 3.java.sql.Statement接口 ###

​	接口的实现在数据库驱动中. 用来操作sql语句，并返回相应结果对象

1. Statement; 执行sql语句对象

- ResultSet  executeQuery(String sql) 根据查询语句返回结果集。只能执行**select**语句。

- int executeUpdate(String sql) 根据执行的DML（insert update delete）语句，返回受影响的行数。

  

- boolean execute(String sql)  此方法可以执行任意sql语句。返回boolean值. 【了解】

  true:  执行select有查询的结果

  false:  执行insert, delete,update

  ​	   执行select没有查询的结果

  



### 4.java.sql.ResultSet接口 ###

1. 封装结果集,查询结果表的对象

- 提供一个游标，默认游标指向结果集第一行之前。
- 调用一次next()，游标向下移动一行。
- 提供一些get方法。

2. ResultSet接口常用API
- boolean next();将光标从当前位置向下移动一行
- int getInt(int colIndex)以int形式获取ResultSet结果集当前行指定列号值
- int getInt(String colLabel)以int形式获取ResultSet结果集当前行指定列名值
- float getFloat(int colIndex)以float形式获取ResultSet结果集当前行指定列号值
- float getFloat(String colLabel)以float形式获取ResultSet结果集当前行指定列名值
- String getString(int colIndex)以String 形式获取ResultSet结果集当前行指定列号值
- String getString(String colLabel)以String形式获取ResultSet结果集当前行指定列名值
- Date getDate(int columnIndex);  以Date 形式获取ResultSet结果集当前行指定列号值
- Date getDate(String columnName);以Date形式获取ResultSet结果集当前行指定列名值


- void close()关闭ResultSet 对象


### 5.小结

#### 5.1API

1. DriverManager 驱动管理器
   + 注册驱动(优化了加载驱动类)
   + 获得连接
2. Connection 连接对象, 操作都是基于这个连接的
   + 创建执行SQL语句对象
3. Statement 执行SQL语句对象
   + executeQuery(String sql); 执行查询 返回结果集
   + executeUpdate(String sql); 执行增删改 返回几行受影响
4. ResultSet 结果集对象
   + next(); 游标向下移动一行, 返回boolean
   + get类型(String 列名); 根据列名获得当前列的数据

#### 5.2注意事项

+ 包名

 ![1554601933427](img/1554601933427.png)





## 四.JDBC操作数据库练习 ##

### 1.单元测试介绍和使用

1. 介绍：JUnit是一个Java语言的单元测试框架。属于第三方工具，一般情况下需要导入jar包，不过，多数Java开发环境			已经集成了JUnit作为单元测试工具

2. 编写测试类，简单理解可以用于取代java的main方法

3. 在测试类方法上添加注解@Test

4. 注解修饰的方法要求：public void 方法名() {…} ，方法名自定义建议test开头，没有参数。

   ![img](img/tu_15.png)

5. 添加IDEA中集成的Junit库，使用快捷键“Alt+Enter”，点击“Add Junit …”

   ![img](img/tu_13.png)


1. 使用：选中方法右键，执行当前方法或者选中类名右键，执行类中所有方法（方法必须标记@Test）

   ![img](img/tu_14.png)


1. 常见使用错误，如果没有添加“@Test”，使用“Junit Test”进行运行，将抛异常

单元测试需要注意的地方:

![img](img/test.bmp)

###2.增删改查 ###

```java
package com.itheima.b_crud;

import com.itheima.bean.User;
import org.junit.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

/**
 * 增删改查
 */
public class CRUDDemo {

    @Test
    //增加  insert into user values(null,'tq','77777','田七');
    public void fun01() throws Exception {
        //1.注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //2.获得连接
        String url = "jdbc:mysql://localhost:3306/web17";
        String user = "root";
        String password = "123456";
        Connection connection = DriverManager.getConnection(url, user, password);

        //3.创建执行sql语句对象
        Statement statement = connection.createStatement();

        //4.执行sql语句
        String sql = " insert into user values(null,'tq','77777','田七')";
        int rows = statement.executeUpdate(sql);
        System.out.println("几行收影响=" + rows);

        //5.释放资源
        if (statement != null) {
            statement.close();
        }
        if (connection != null) {
            connection.close();
        }

    }

    @Test
    //更新 把id为5的用户的密码改成88888888
    public void fun02() throws Exception {
        //1.注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //2.获得连接
        String url = "jdbc:mysql://localhost:3306/web17";
        String user = "root";
        String password = "123456";
        Connection connection = DriverManager.getConnection(url, user, password);

        //3.创建执行sql语句对象
        Statement statement = connection.createStatement();

        //4.执行sql语句
        String sql = "update user set  password = '88888888' where id = 5";
        statement.executeUpdate(sql);

        //5.释放资源
        if (statement != null) {
            statement.close();
        }
        if (connection != null) {
            connection.close();
        }

    }

    @Test
    //删除id为5的用户
    public void fun03() throws Exception {
        //1.注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //2.获得连接
        String url = "jdbc:mysql://localhost:3306/web17";
        String user = "root";
        String password = "123456";
        Connection connection = DriverManager.getConnection(url, user, password);

        //3.创建执行sql语句对象
        Statement statement = connection.createStatement();

        //4.执行sql语句
        String sql = "delete from user where id = 5";
        statement.executeUpdate(sql);

        //5.释放资源
        if (statement != null) {
            statement.close();
        }
        if (connection != null) {
            connection.close();
        }
    }

    @Test
    //查询id为1的用户
    public void fun04() throws Exception {
        //1.注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //2.获得连接
        String url = "jdbc:mysql://localhost:3306/web17";
        String username = "root";
        String password = "123456";
        Connection connection = DriverManager.getConnection(url, username, password);

        //3.创建执行sql语句对象
        Statement statement = connection.createStatement();

        //4.执行sql语句
        String sql = "select * from user where id = 1";
        ResultSet resultSet = statement.executeQuery(sql);

        User user = null;
        while (resultSet.next()) {
            //每遍历一次,就是一条数据.就是一个User对象(有数据才有user)
            user = new User(resultSet.getInt("id"),
                    resultSet.getString("username"),
                    resultSet.getString("password"),
                    resultSet.getString("nickname"));
        }


        //获得用户名
        System.out.println("用户名="+user.getUsername());

        //5.释放资源
        if (resultSet != null) {
            resultSet.close();
        }

        if (statement != null) {
            statement.close();
        }
        if (connection != null) {
            connection.close();
        }
    }

    @Test
    //查询所有用户
    public void fun05() throws Exception {
        //1.注册驱动
        //DriverManager.registerDriver(new Driver());
        //类全限定名(带包名), 加载Driver类, 静态代码块就会执行, 驱动就注册了
        Class.forName("com.mysql.jdbc.Driver");

        //2.获得连接(连接数据库)
        //连接数据库路径
        String url="jdbc:mysql://localhost:3306/web17";
        String username = "root";
        String password= "123456";
        Connection connection = DriverManager.getConnection(url, username, password);

        //3.创建执行sql语句的对象
        Statement statement = connection.createStatement();

        //4.执行sql语句, 处理结果
        String sql = "select * from user";
        ResultSet resultSet = statement.executeQuery(sql);

        List<User> list = new ArrayList<User>();

        while (resultSet.next()){
            //每遍历一次就是一条数据, 就封装成一个User对象. 把封装的每一个User添加到list集合里面
            User user = new User(resultSet.getInt("id"),
                    resultSet.getString("username"),
                    resultSet.getString("password"),
                    resultSet.getString("nickname")
            );
            list.add(user);
        }


        //获得第二个用户的用户名
        System.out.println(list.get(1).getUsername());

        //5.释放资源(先创建的后关闭)
        if(resultSet != null){
            resultSet.close();
        }

        if(statement != null){
            statement.close();
        }
        if(connection != null){
            connection.close();
        }

    }


}

```


### 3.JDBC工具类的抽取 ###

- 创建配置文件,配置文件在src目录下,扩展名是properties

  ![1537261759376](img/1537261759376.png)

  配置文件:

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/web17
username=root
password=123456
```

​		


-  工具类实现

   

   ```java
   package com.itheima.utils;

   import java.io.FileInputStream;
   import java.io.IOException;
   import java.io.InputStream;
   import java.sql.*;
   import java.util.Properties;

   /**
   * JDBC工具类
   * 1.获得连接和释放资源
   * 2. 驱动只需要注册一次
   */
   public class JdbcUtils {

      private static String driver;
      private static String url;
      private static String username;
      private static String password;

      //JdbcUtil一加载,静态代码块就会执行, 只执行一次
      static {

          InputStream is = null;
          try {
              //0 加载配置文件
              //a. 读取jdbc.properties
               is = JdbcUtils.class.getClassLoader().getResourceAsStream("jdbc.properties");
              //b.创建Properties对象
              Properties properties = new Properties();
              properties.load(is);
              //c .读取值
               driver = properties.getProperty("driver");
               url = properties.getProperty("url");
               username = properties.getProperty("username");
               password = properties.getProperty("password");
              //1.注册驱动
              Class.forName(driver);
          } catch (Exception e) {
              e.printStackTrace();
          }finally {
              if (is != null){
                  try {
                      is.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
      /**
       * 获得连接
       * @return
       * @throws Exception
       */
      public static Connection getConnection() throws Exception {
          //2.获得连接(连接数据库)
          Connection connection = DriverManager.getConnection(url, username, password);
          return connection;
      }
     
       /**
      * 释放资源
      *
      * @param resultSet
      * @param statement
      * @param connection
      */
     public static void release(ResultSet resultSet, Statement statement, Connection connection) {

         if (resultSet != null) {
             try {
                 resultSet.close();
             } catch (SQLException e) {
                 e.printStackTrace();
             }
         }

         if (statement != null) {
             try {
                 statement.close();
             } catch (SQLException e) {
                 e.printStackTrace();
             }
         }
         if (connection != null) {
             try {
                 connection.close();
             } catch (SQLException e) {
                 e.printStackTrace();
             }
         }

     }
   }
   ```












## 五,实现一个用户登录功能 ##

### 1.需求

​	在控制台输入用户名和密码,查询数据库,如果数据库存在当前用户,显示登录成功!

​	如果数据库不存在当前用户,显示登录失败!																		

![img](img/tu_8.gif)



### 2分析

![1537262185232](img/1537262185232.png)



1. 获得用户输入的用户名和密码
2. 使用JDBC, 根据用户名和密码查询数据库, 封装成User对象
3. 判断是否登录成功(说白了就是判断User对象是为null) 
   + user不为null, 给用户提示 登录成功
   + user为null, 给用户提示 登录失败



### 3.代码实现

+ LoginClient.java

```java
package com.itheima.b_login;

import com.itheima.bean.User;
import com.itheima.util.JdbcUtil;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.Scanner;

/**
 * @Description: 登录案例
 * @Author: yp
 */
public class LoginClient {

    public static void main(String[] args) throws Exception {
        // 1. 获得用户输入的用户名和密码
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入用户名:");
        String username = scanner.nextLine();
        System.out.println("请输入密码:");
        String password = scanner.nextLine();

        // 2. 使用JDBC, 根据用户名和密码查询数据库, 封装成User对象
        //a. 获得连接(工具类)
        Connection connection = JdbcUtil.getConnection();
        //b.创建执行sql语句对象
        Statement statement = connection.createStatement();
        //c.执行sql语句
        String sql = "SELECT * FROM user WHERE username  ='" + username + "' AND password ='" + password + "'";
        ResultSet resultSet = statement.executeQuery(sql);
        //d.处理
        User user = null;
        while (resultSet.next()) {
            user = new User(resultSet.getInt("id"),
                    resultSet.getString("username"),
                    resultSet.getString("password"),
                    resultSet.getString("nickname")
            );
        }

        // 3. 判断是否登录成功(说白了就是判断User对象是为null)
        if (user != null) {
            //  user不为null, 给用户提示 登录成功
            System.out.println("登录成功!尊敬的" + user.getNickname());
        } else {
            // user为null, 给用户提示 登录失败
            System.err.println("登录失败");
        }

        //4. 释放资源
        JdbcUtil.closeAll(resultSet,statement,connection);
    }

}
```



### 4.登录存在安全问题

+ 输入的密码  ' or '' = ', 语句如下

```
SELECT * FROM user WHERE username  ='zs' AND password ='' or ''=''
SELECT * FROM user WHERE username  ='zs' AND password ='' or TRUE
SELECT * FROM user WHERE  TRUE
SELECT * FROM user
```

+ 发现语句出现了sql注入问题

  把用户输入的 or 当成==关键词== 注入到了sql语句里面了

## 六,SQL注入问题解决：preparedStatement ##

### 1.preparedStatement概述    

预编译SQL语句对象， 是Statement对象的子接口。

特点：

- 性能要高
- 会把sql语句先编译,格式固定好,
- sql语句中的参数会发生变化，过滤掉用户输入的关键字(eg: or)


### 2.用法

#### 2.1通过connection对象创建

- connection.prepareStatement(String sql) ;创建preparedStatement对象

- sql表示预编译的sql语句,如果sql语句有参数通过?来占位

  ```
  SELECT * FROM user where username = ? AND password =?
  ```

#### 2.2过setxxx方法来指定参数

- preparedStatement.setxxx(int i,Object obj); i 指的就是问号的索引(指第几个问号,从1开始),xxx是类型(eg:int,String,Long)

#### 2.3实例

```java
    //b.创建预编译的SQL语句对象(SQL参数需要使用?占位)
        String sql = "SELECT * FROM user WHERE username  = ? AND password = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //c.设置参数, 执行(还是executeQuery()和executeQUpdate(), 但是不需要再传入SQL语句, 上面已经传入了)
        preparedStatement.setString(1,username);
        preparedStatement.setString(2,password);
        ResultSet resultSet = preparedStatement.executeQuery();
```
### 3.使用preparedStatement改写CURD练习



```java
**
 * 增删改查 ?只能占参数, sql语句里面的表名, 关键词(select,insert等等 )都不可以占
 */
public class CRUDDemo {

    @Test
    //增加
    public void fun01() throws Exception {
        Connection connection = JdbcUtils.getConnection();

        //1.创建预编译sql语句对象
        String sql = "insert into user values(null,?,?,?)";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //设置参数?
        preparedStatement.setString(1,"wb");
        preparedStatement.setString(2,"88888");
        preparedStatement.setString(3,"王八");

        //2.执行
        preparedStatement.executeUpdate();

        //3释放资源
        JdbcUtils.release(null,preparedStatement,connection);


    }

    @Test
    //更新 把id为7的用户的密码改成999999; update user set password  = '9999999' where id = 7
    public void fun02() throws Exception {
        Connection connection = JdbcUtils.getConnection();

        //1.创建预编译sql语句对象
        String sql = "update user set password  = ? where id = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //设置参数?
        preparedStatement.setString(1,"99999999");
        preparedStatement.setInt(2,7);

        //2.执行
        preparedStatement.executeUpdate();

        //3释放资源
        JdbcUtils.release(null,preparedStatement,connection);


    }

    @Test
    //删除id为7的用户
    public void fun03() throws Exception {

        Connection connection = JdbcUtils.getConnection();

        //1.创建预编译sql语句对象
        String sql = "DELETE FROM user WHERE id = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //设置参数
        preparedStatement.setInt(1,7);

        //2.执行
        preparedStatement.executeUpdate();

        //3释放资源
        JdbcUtils.release(null,preparedStatement,connection);

    }

    @Test
    //查询id为1的用户
    public void fun04() throws Exception {
        Connection connection = JdbcUtils.getConnection();

        //1.创建预编译sql语句对象
        String sql = "SELECT * FROM  user WHERE  id  = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //设置参数?
        preparedStatement.setInt(1,1);

        //2.执行
        ResultSet resultSet = preparedStatement.executeQuery();
        User user = null;
        while(resultSet.next()){
            user = new User(resultSet.getInt("id"),
                    resultSet.getString("username"),
                    resultSet.getString("password"),
                    resultSet.getString("nickname"));
        }

        System.out.println(user);

        //3释放资源
        JdbcUtils.release(resultSet,preparedStatement,connection);

    }

    @Test
    //查询所有用户
    public void fun05() throws Exception {

        Connection connection = JdbcUtils.getConnection();

        //1.创建预编译sql语句对象
        String sql = "select * from user";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

        //2.执行
        ResultSet resultSet = preparedStatement.executeQuery();
        List<User> list = new ArrayList<User>();

        while(resultSet.next()){
            User user = new User(resultSet.getInt("id"),
                    resultSet.getString("username"),
                    resultSet.getString("password"),
                    resultSet.getString("nickname"));
            list.add(user);
        }

        System.out.println(list);
        //3释放资源
        JdbcUtils.release(resultSet,preparedStatement,connection);


    }


}
```



### 4.小结

#### 4.1用法

+ 获得, 参数就是一个预编译的SQL语句, 如果有参数通过?代替

```
PreparedStatement preparedStatement = connection.prepareStatement(sql);
```

+ 设置参数

```
preparedStatement.set类型(int ?的索引, Object 值)
```

+ 执行, 不传入sql语句的

```
preparedStatement.executeQuery(); 
preparedStatement.executeUpdate(); 
```

#### 4.2注意

+ ?只能占列的值,表名,关键字都是不可以进行占位



## 总结

### 1.JDBC概念

+ 什么是JDBC
+ 为什么要学习JDBC

### 2.JDBC步骤

+ 加载驱动
+ 获得连接
+ 创建预编译的SQL语句对象
+ 设置参数, 执行
+ 释放资源

### 3.使用PrepareStatement进行CRUD

+ 根据JDBC步骤练习

### 4.抽取工具类

### 5.登录案例

