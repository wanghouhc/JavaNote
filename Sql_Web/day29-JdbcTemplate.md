---
typora-copy-images-to: img
---

# day19-JdbcTemplate

# 学习目标

- [ ] 掌握JdbcTemplate实现增删改
- [ ] 掌握JdbcTemplate实现查询
- [ ] 能够说出什么是数据库元数据
- [ ] 掌握自定义数据库框架，实现增加、删除、更新方法
- [ ] 能够理解分层的作用

# 第一章-使用jdbcTemplate完成增删改查的操作

## 知识点-JavaBean

### 1.目标

+ 掌握JavaBean的编写要求, 区分字段和JavaBean属性

### 2.路径

1. JavaBean编写要求
2. 字段和属性区别

### 3.讲解

#### 3.1JavaBean编写要求

JavaBean就是一个类，在开发中常用于封装数据。只是有一些特殊需求：

1. ==提供私有字段==
   - `private 类型 字段名;`
2. ==提供公共 getter/setter方法==
   - `public void setXXX(){}`
3. ==提供无参构造==
   - 如果写了含参构造器，则无参构造器一定要写
4. ==需要实现序列化接口==
   - `class 类名 implements java.io.Serializable `

#### 3.2 字段和属性

​	字段：就是成员变量，字段名就是成员变量名。

​	属性：属性名把setter/getter方法的set/get前缀去掉后，剩下的内容首字母小写获得。  

示例： `getId()`  ---》 去掉get/set前缀后得到`Id`  --> 首字母小写得到 `id`（即为属性）

大都数情况下，IDEA生成get、set方法，

- ```java
  public Integer getId() {
          return id;
  }
  
  public void setId(Integer id) {
      this.id = id;
  }
  ```

- 一般情况下, ==习惯通过IDEA生成, 把字段和属性设置成一样的==

```java
package com.itheima.bean;

import java.io.Serializable;

public class User implements Serializable {
    // 1. 字段私有化
    private Integer id;
    private String username;
    private String password;
    private String nickname;

    // 2. 公有的set、get方法
    public Integer getId() {
        return id;
    }

    public void setId01(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    // 3.必须提供空构造器
    public User() {
    }

    public User(Integer id, String username, String password, String nickname) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.nickname = nickname;
    }
}

```

### 4.小结

- javabean的要求
  - 字段私有
  - 公共的get、set方法
  - 无参构造器
  - 实现序列化接口
- 字段、属性
  - 字段： 就是类的成员变量
  - 属性： get、set方法去掉get、set前缀，剩下的内容首字母改为小写：  setId()-----> Id--->id属性

## 知识点-jdbcTemplate介绍

### 1.目标

+ 知道什么是jdbcTemplate和基本核心API

### 2.路径

1. jdbcTemplate概述
2. jdbcTemplate核心API

### 3.讲解

#### 3.1概述

​	JdbcTemplate 就是Spring框架里面对JDBC的封装，是一个工具包. 目的使用jdbc操作数据库步骤更加的简洁。JdbcTemplate是Spring的一部分。

JdbcTemplate 为什么这么简洁？

- JdbcTemplate处理了资源的建立和释放。

- JdbcTemplate 帮助我们避免一些常见的错误，比如忘了总要关闭连接。

- JdbcTemplate运行核心的JDBC工作流，如PreparedStatement的建立和执行都帮我们做了，而我们只需要提供SQL语句和提取结果。 

#### 3.2jdbcTemplate核心API

| 方法                                       | 作用                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| public JdbcTemplate(DataSource dataSource) | 构造方法，传递数据源做为参数![1563848623186](img\1563848623186.png) |
| int update(String sql, Object…args)        | DML操作---增加、删除、修改使用的方法；args可以传入多个参数   |
| queryForMap()                              | 返回Map<String,Object>的查询结果，其中键是列名，值是表中对应的记录。 |
| queryForObject()                           | 查询一个对象                                                 |
| queryForList()                             | 返回多条记录的查询结果，封装成一个List集合,但是封装的是Map   |
| query()                                    | 通用的查询方法，有多个同名方法的重载，可以自定义查询结果集封装成什么样的对象。 |

### 4.小结

- JdbcTemplate
  - 是什么？ spring框架的一部分，一个工具包； 使得jdbc操作数据库的步骤更少了
- API
  - 构造器需要传入一个连接池对象作为参数
  - DML操作： update
  - DQL： queryXX()

## 案例-使用jdbcTemplate完成CRUD

### 1.需求

+ 使用jdbcTemplate完成CRUD

### 2.分析

1. 创建项目,导入jar

![img](img/tu_8.png)

2. 创建jdbctemplate对象， 需要传入连接池对象作为参数
3. 使用update、queryXX方法

### 3.实现

#### 3.1JdbcTemplate实现增删改

##### 3.1.1API介绍

```java
public int update(String sql, Object ... params);  //DML操作---增加、删除、修改使用的方法；params可以传入多个参数
```

##### 3.1.2代码实现

- 增加

```java
@Test
    public void test01(){
        // 1. 创建JdbcTemplate对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate(C3p0Util.getDataSource());

        // 2. 插入一个用户
        String sql = "insert into user values(?,?,?,?)";
        Object[] params = new Object[]{null, "韩信", "111", "跳跳"};
        int update = jdbcTemplate.update(sql, params);
        System.out.println(update);
    }
```

+ 更新

```java
// 修改用户的密码
    @Test
    public void test02(){
        // 1. 创建JdbcTemplate对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate(C3p0Util.getDataSource());

        // 2. 调用API
        String sql = "update user set password = ?  where username = ?";
        Object[] params = {"666", "韩信"};
        jdbcTemplate.update(sql, params);
    }
```

- 删除

```java
// 删除用户
    @Test
    public void test03(){
        // 1. 创建JdbcTemplate对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate(C3p0Util.getDataSource());

        // 2. API
        String sql = "delete from user where id = ?";
        Object[] params = {7};
        jdbcTemplate.update(sql, params);
    }
```

- 使用JdbcTemplate进行DML操作
  - JdbcTemplate(DataSource对象作为参数)
  - 使用update(sql, args...)



#### 3.2JdbcTemplate实现查询

##### 3.2.1查询一条记录封装成Map

- API介绍：
  - ==Map<String, Object> queryForMap(String sql, Object ... params)==
  - 将查询结果集封装成map集合
  - 示意图
  - ![1571194527822](img\1571194527822.png)

+ 开发步骤:
+ 创建JdbcTemplate对象
  + 使用API

- 需求: 查询id为1的用户, 封装成map对象

+ 代码实现

```java
 //1. 创建JdbcTemplate对象(需要传入连接池)
 JdbcTemplate jdbcTemplate = new JdbcTemplate(C3P0Utils.getDataSource());

//2. 执行queryForMap()的方法来查询
 String sql = "select * from user where id = ?";
Map<String, Object> map = jdbcTemplate.queryForMap(sql, 1);
```

##### 3.2.2查询一条记录封装成实体对象(bean)

- **RowMapper**接口介绍
  
  - 这是表示行记录和对象的映射规范
  
  - 常用的实现类 **BeanPropertyRowMapper**： 代表bean的属性和行记录的列进行匹配映射
  
    ```java
    class BeanPropertyRowMapper<T> implements RowMapper<T>{
        ...
    }
    ```
  
- API介绍

  - ==类 queryForObject(String sql, new BeanPropertyRowMapper<>(类.class), Object[] params);==
  - 会将查询到的一条记录封装成你想要的目标类型对象返回：
    - 如果使用 `User queryForObject(String sql, new BeanPropertyRowMapper<>(User.class), Object[] params); ` 会自动封装所有和列同名的==属性值==，从而得到的就是一个User对象
  - 画图示意
  - ![1571194891176](img\1571194891176.png)

+ 需求: 查询id为1的用户, 封装成user对象

+ 开发步骤

  ​	 创建JdbcTemplate对象，传入数据源

  ​	 编写查询的SQL语句

  ​	 使用JdbcTemplate对象的==queryForObject==方法，并传入需要返回的数据的类型   返回是一个实体对象

+ 代码实现

```java
 //1. 创建JdbcTemplate对象(需要传入连接池)
 JdbcTemplate jdbcTemplate = new JdbcTemplate(C3P0Utils.getDataSource());

  //2. 执行queryForObject(sql,封装器,参数)的方法来查询;
 //要求列名必须和JavaBean属性一致, 如果不一致是不能封装的
 String sql = "select * from user where id = ?";
 User user = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(User.class), 1);
```

##### 3.2.3查询多条记录封装成 `List<Map<String,Object>>`

- API介绍

  - `List<Map<String,Object>> queryForList(String sql, Object... params)`
  - 得到一个List集合，这个集合里面存放的都是 Map<String, Object>
  - 画图示意
  - ![1571195623376](img\1571195623376.png)

- 需求: 查询所有的用户, 封装成 `List<Map<String,Object>> list`

- 开发步骤:

  ​	 创建JdbcTemplate对象，传入数据源

  ​	  编写SQL语句

  ​	  使用JdbcTemplate对象的==queryForList(String sql,Objec....params)==方法查询结果

- 代码实现

```java
//1. 创建JdbcTemplate对象(需要传入连接池)
JdbcTemplate jdbcTemplate = new JdbcTemplate(C3P0Utils.getDataSource());

//2. 调用queryForList()查询,一条数据就对应一个map, 再把多个map封装成到List集合里面
String sql = "select * from user";
 List<Map<String, Object>> mapList = jdbcTemplate.queryForList(sql);
```

##### 3.2.4查询多条记录封装成 `List<JavaBean>`

- API介绍

  - `List<类> query(String sql, new BeanPropertyRowMapper<>(类.class), Object... params)`
  - 就是将查询到的多条记录封装成你想要的数据类型的List集合返回
  - 画图示意
  - ![1571195945592](img\1571195945592.png)

- 需求: 查询所有的用户, 封装成 `List<JavaBean> list`

- 开发步骤:

  ​	 创建JdbcTemplate对象，传入数据源

  ​	  编写SQL语句

  ​	  使用JdbcTemplate对象的==query(String sql,new BeanPropertyRowMapper<>,Object .... params)==方法查询结果

- 代码实现

```java
//1. 创建JdbcTemplate对象(需要传入连接池)
JdbcTemplate jdbcTemplate = new JdbcTemplate(C3P0Utils.getDataSource());

//2. 调用query(sql, 封装器)查询,一条数据就对应一个User对象, 再把多个对象封装成到List集合里面
String sql = "select * from user";
List<User> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(User.class));
```

##### 3.2.5统计总记录数

- 需求: 统计user的总记录数

- 开发步骤:

  ​	 创建JdbcTemplate对象，传入数据源

  ​	  编写SQL语句

  ​	  使用JdbcTemplate对象的使用==queryForObject()==方法，指定参数为Integer.class或者Long.class

- 代码实现

```java
 //1. 创建JdbcTemplate对象(需要传入连接池)
JdbcTemplate jdbcTemplate = new JdbcTemplate(C3P0Utils.getDataSource());

//2. 调用queryForObject(Long.class) 查询
String sql = "select count(*) from user";
Long count = jdbcTemplate.queryForObject(sql, Long.class);
```

### 4.小结

- 单条记录
  - 封装成Map<String , Object>  :     queryForMap(sql, params)
  - 封装成javabean  :     queryForObject(sql,  new BeanpropertyRowMapper<>(类型.class), params)
- 多条记录
  - List<Map<String , Object>> :  queryForList(sql,  params)
  - List<javabean>  :  query(sql, new BeanpropertyRowMapper<>(类型.class), params)
- 统计个数
  - queryForObject(sql, Integer/Long.class)

# 第二章-自定义JDBCTemplate(增删改)

## 知识点-元数据

### 1.目标

+ 能够说出什么是数据库元数据

### 2.分析

​	我们要自定义JDBCTemplate, 就需要知道列名, 参数个数等, 这些可以通过数据库的元数据库进行获得.元数据在建立框架和架构方面是特别重要的知识,我们可以使用数据库的元数据来创建自定义JDBC框架, 模仿jdbcTemplate.

### 3.讲解

#### 3.1什么是元数据

​	==元数据(MetaData)，即定义数据的数据。==

​	示例：打个比方，就好像我们要想搜索一首歌(歌本身是数据)，而我们可以通过歌名，作者，专辑等信息来搜索，那么这些歌名，作者，专辑等等就是这首歌的元数据。

![1571196736005](img\1571196736005.png)

因此==数据库的元数据就是一些注明数据库信息的数据。==（数据库、表、列...）

​	简单来说: **数据库的元数据就是 数据库、表、列的定义信息。**

​		① 由PreparedStatement对象的元数据：  占位符的参数个数...

　　② 由ResultSet对象的元数据： 列名、值、数据类型...

#### 3.2 PreparedStatement 的元数据

##### 3.2.1概述

预编译语句对象的元数据主要包括==参数的个数==、参数的类型(mysql不支持)，是一个 `ParameterMetaData` 类型， 可以通过以下方法获得：

- ```java
  ParameterMetaData parameterMetaData = preparedStatement.getParameterMetaData();
  ```

##### 3.2.2 ParameterMetaData相关的API

- ==int getParameterCount();==   可以获得预编译语句对象中占位符的参数个数
- int getParameterType(int param) 获取指定参数的SQL类型。 (注:MySQL不支持获取参数类型)

##### 3.2.3实例代码

```java
//3. 获得参数的 元数据
ParameterMetaData parameterMetaData = preparedStatement.getParameterMetaData();
// 获得sql语句里面参数的个数
int parameterCount = parameterMetaData.getParameterCount();
System.out.println("parameterCount="+parameterCount);
```

#### 3.3 ResultSet 的元数据

##### 3.3.1概述

 查询结果集的元数据，类型为 `ResultSetMetaData`可用于获取有关`ResultSet`对象中列的类型和属性的信息。

- 通过==ResultSetMetaData==可以获得结果集的元数据：
  - 列的个数
  - 列名
  - 列的sql类型
  - 列的java类型

![img](img\1571205183856.png)

如何获得ResultSetMetaData？

 	==ResultSetMetaData resultSetMetaData =  resultSet.getMetaData()==

##### 3.3.2 resultSetMetaData 相关的API

- `getColumnCount();`   获取结果集中列项目的个数
- `getColumnName(int column);`   获得数据指定列的列名
- `getColumnTypeName(); `    获取指定列的SQL类型
- `getColumnClassName(); `  获取指定列SQL类型对应于Java的类型

##### 3.2.3实例代码

```java
       //4. 获得结果的元数据对象
        ResultSetMetaData resultSetMetaData = resultSet.getMetaData();

        // 获得结果集里面列的数量
        int columnCount = resultSetMetaData.getColumnCount();
        System.out.println("columnCount="+columnCount);//4

        // 获得某一列的列名 eg: 获得第二列
        String columnName = resultSetMetaData.getColumnName(2);
        System.out.println("columnName="+columnName);

        // 获得第三列的sql类型 varchar
        String columnTypeName = resultSetMetaData.getColumnTypeName(3);
        System.out.println(columnTypeName);

        // 获得第三列对应的java类型 String
        String columnClassName = resultSetMetaData.getColumnClassName(3);
        System.out.println("columnClassName="+columnClassName);
```

#### 4.小结

- 预编译语句对象的元数据
  - 获得占位符参数的个数
- 结果集元数据
  - 列的个数
  - 列名
  - 列sql类型
  - 列java类型



## 案例-自定义JDBCTemplate

### 1.需求

+ 模仿JDBCTemplate, 完成增删改的功能

### 2.分析

#### 2.1整体结构

![](img\1571207017274.png)

#### 2.2内部编码(update里面)

![1571207036504](img\1571207036504.png)

### 3.实现

```java
package com.itheima.c_jdbcTemplate;

import com.itheima.util.C3p0Util;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ParameterMetaData;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class MyJdbcTemplate {

    private DataSource dataSource;
    // 1. 构造器，参数为数据源对象
    public MyJdbcTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }


    // 2. 定义update方法-->Object[] params占位符的值
    public int update(String sql, Object[] params)  {

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {
            // 1. 获得连接
            connection = dataSource.getConnection();

            // 2. 创建语句对象
            preparedStatement = connection.prepareStatement(sql);

            // 3. 设置值
            // 元数据--->获取表里面的属性个数
            ParameterMetaData parameterMetaData = preparedStatement.getParameterMetaData();
            int parameterCount = parameterMetaData.getParameterCount();
            for (int i = 1; i <= parameterCount; i++){
                preparedStatement.setObject(i, params[i-1]);
            }

            // 执行
            int row = preparedStatement.executeUpdate();
            // 返回影响的行数
            return  row;
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                C3p0Util.closeAll(null, preparedStatement, connection );
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        return -1;
    }
}

```

### 4.小结

- 模仿JdbcTemplate
  - 构造器
  - update方法

# 第三章-三层架构入门

## 案例-完成注册案例

### 1.需求

​	获得用户的输入,  最终把数据存到数据库. 就是向数据库里面存一条数据 

生活中的注册页面：

![1571225767849](img\1571225767849.png)

### 2案例思路 

- 获得用户输入的用户名,密码,昵称
- 使用JDBCTemplate操作数据库 
- 判断是否注册成功, 给用户提示

### 3.代码实现

```java
public class UserClient {

    public static void main(String[] args) {
        //1.获得用户输入的用户名,密码,昵称
        Scanner scanner = new Scanner(System.in);
        System.out.println("用户名:");
        String username = scanner.nextLine();
        System.out.println("密码:");
        String password = scanner.nextLine();
        System.out.println("昵称:");
        String nickname = scanner.nextLine();

        //2.使用JDBCTemplate操作数据库

        JdbcTemplate jdbcTemplate = new JdbcTemplate(C3P0Utils.getDataSource());
        //去查询数据库 用户名对应的user是否存在


        String sql = "insert into user values(?,?,?,?)";
        int rows = jdbcTemplate.update(sql, null, username, password, nickname);
        //3.判断是否注册成功, 给用户提示
        if(rows > 0){
            System.out.println("注册成功!请登录...");
        }else{
            System.out.println("注册失败!");
        }


    }
}
```

### 4.小结

1. 获取用户输入的信息
2. 使用JdbcTemplate插入数据库
3. 如果影响行数>0表示注册成功



## 知识点-分层的作用

### 1.目标

+ 体会分层的好处

### 2.分析

​	我们之前的注册案例是将用户输入，数据库的操作，逻辑处理放在了同一个方法中，这样虽然非常直观，但是等项目做大的时候非常不好维护代码，也不好增加功能 

### 3.讲解



后端工程师

前端工程师

安卓

IOS

测试

产品经理

运维网络

运营（业务）



+ 软件中分层：按照不同功能分为不同层，通常分为三层：表现层(web层)，业务层，持久(数据库)层。

  

  ![img](img/tu_12.png)

+ 不同层次包名的命名


| 分层                       | 包名(公司域名倒写)          |
| ------------------------ | ------------------- |
| 表现层(web层)  注: 后续学javaweb | com.itheima.web     |
| 业务层(service层)            | com.itheima.service |
| 持久层(数据库访问层)              | com.itheima.dao     |
| JavaBean                 | com.itheima.bean    |
| 工具类                      | com.itheima.utils   |

+ 分层的意义:
  1. 解耦：降低层与层之间的耦合性。
  2. 可维护性：提高软件的可维护性，对现有的功能进行修改和更新时不会影响原有的功能。
  3. 可扩展性：提升软件的可扩展性，添加新的功能的时候不会影响到现有的功能。
  4. 可重用性：不同层之间进行功能调用时，相同的功能可以重复使用。

### 4.使用分层改写注册案例

+ UserClient

```java
package com.itheima.f_regist;

import com.itheima.bean.User;
import com.itheima.util.C3p0Util;
import org.springframework.jdbc.core.JdbcTemplate;

import java.util.Scanner;

public class UserClient {
    public static void main(String[] args) {
        // 1.获取输入信息
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入用户名：");
        String username = scanner.nextLine();

        System.out.println("请输入密码：");
        String password = scanner.nextLine();

        System.out.println("请输入昵称：");
        String nickname = scanner.nextLine();


        UserService userService = new UserService();
        // 调用业务层
        // 调用谁就是让谁去干
        User user = new User(null, username, password, nickname);
        boolean flag = userService.regist(user);

        // 3.反馈信息
        if(flag){
            System.out.println("注册成功");
        }else{
            System.out.println("注册失败");
        }



    }
}

```

+ UserService

```java
package com.itheima.f_regist;

import com.itheima.bean.User;

public class UserService {
    public boolean regist(User user) {
        // 验证码、邮箱激活
        // TODO
        // 保存信息
        UserDao userDao = new UserDao();
        int row = userDao.save(user);
        /*if(row > 0){
            return  true;
        }else {
            return false;
        }*/
        return row > 0;
    }
}

```

+ UserDao

```java
package com.itheima.f_regist;

import com.itheima.bean.User;
import com.itheima.util.C3p0Util;
import org.springframework.jdbc.core.JdbcTemplate;

public class UserDao {
    public int save(User user) {

        JdbcTemplate jdbcTemplate = new JdbcTemplate(C3p0Util.getDataSource());
        String sql = "insert into user values(?,?,?,?)";
        Object[] params = new Object[]{null, user.getUsername(), user.getPassword(), user.getNickname()};
        int update = jdbcTemplate.update(sql, params);
        return update;
    }
}

```



### 5.小结

1. 三层架构实现注册案例
   1. web层
      1. 获取参数
      2. 调用业务层
      3. 做出响应
   2. 业务层service层
      1. 处理业务
      2. 调用dao层
   3. 持久层dao
      1. 和数据库打交道



# 总结

### JdbcTemplate

- JdbcTemplate是什么？   spring框架的一个工具包，可以简化jdbc操作数据库的步骤
- javabean
  - 字段私有
  - 公共的get、set方法
  - 无参构造器
  - 实现序列化接口
- 字段、属性
  - 字段： 成员变量
  - 属性：  get、set方法去掉前缀，剩下的内容首字母小写：  getId()---》Id--->  id
- API
  - DML操作：  update(sql, params)
  - DQL:  queryXX();
- 结果集是一个记录
  - 封装成 Map:   queryForMap(sql, params)
  - 封装成javabean:  javabean对象 queryForObject(sql, new BeanPropertyRowMapper<>(类型.class), params)
- 结果集是多个记录
  - 封装成 List<Map>:   queryForList(sql, params)
  - 封装成List<javabean>: query(sql, new BeanPropertyRowMapper<>(类型.class), params)
- 获取个数
  - queryForObject(sql, Integer.class)





### 三层架构

几层：

- web层、表现层
- 业务层service
- 持久层dao



作用：

- 解耦
- 可扩展性
- 可维护性
- 可重用性

