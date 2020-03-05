# day01-MyBatis

# 学习目标

+ 能够了解什么是框架
+ 掌握Mybatis框架开发快速入门
+ 掌握Mybatis框架的基本CRUD操作
+ 掌握Mybatis的参数深入
+ 掌握SqlMapConfig.xml配置文件 

# 第一章-框架概述 

## 知识点-框架概述

### 1.目标

- [ ] 能够了解什么是框架

### 2.路径

1.讲解什么是框架

2.框架要解决的问题

3.分层架构下的常见框架

### 3.讲解

#### 3.1什么是框架 

​	   框架（Framework）是整个或部分系统的可重用设计，表现为一组抽象构件及构件实例间交互的方法;另一种定义认为，框架是可被应用开发者定制的应用骨架。前者是从应用方面而后者是从目的方面给出的定义。

简而言之，**框架其实就是某种应用的半成品，就是一组组件，供你选用完成你自己的系统。简单说就是使用别人搭好的舞台，你来做表演。而且，框架一般是成熟的，不断升级的软件**。

之前所学的持久城框架JdbcTemplate就属于一个框架，它封装了Jdbc代码，我们对比下封装前和封装后的操作实现。

​		框架：框架的本质就是将可重用的代码进行封装成一组组件，通过调用它暴露的api，就可以组合使用这些组件，简单高效的完成项目的开发。

​		**JDBC代码**

![1558318496903](D:/mybatis/%E7%AC%AC1%E5%A4%A9/%E8%AE%B2%E4%B9%89/images/1558318496903.png)



**JdbcTemplate**

![1558318595942](D:/mybatis/%E7%AC%AC1%E5%A4%A9/%E8%AE%B2%E4%B9%89/images/1558318595942.png)

相同的操作，使用JdbcTemplate就非常简单了，不用关注那些大量的非业务逻辑代码，只需要关注自己的业务实现即可。

#### 3.2框架要解决的问题 

​	   框架要解决的最重要的一个问题是**技术整合**的问题，在 J2EE 的 框架中，有着各种各样的技术，不同的软件企业需要从J2EE 中选择不同的技术，这就使得软件企业最终的应用依赖于这些技术，技术自身的复杂性和技术的风险性将会直接对应用造成冲击。而应用是软件企业的核心，是竞争力的关键所在，因此应该将应用自身的设计和具体的实现技术解耦。这样，软件企业的研发将集中在应用的设计上，而不是具体的技术实现，技术实现是应用的底层支撑，它不应该直接对应用产生影响。

​	 框架一般处在低层应用平台（如 J2EE）和高层业务逻辑之间的中间层。

​	框架要解决的问题:

​				框架的开发者：将底层技术和上层应用解耦。

​				框架的使用者：框架的整合问题。

#### 3.3分层开发下的常见框架 

​	通过分层更好的实现了各个部分的职责，在每一层将再细化出不同的框架，分别解决各层关注的问题。![img](img/tu_14.png)

### 4.小结

​	1.什么是框架

​			就是把重用代码进行封装为一组组件，框架提供api，将组件结合使用，从而达到简化开发的目的。

​	2.框架解决的问题

​			框架的开发人员：将底层的技术和上层应用解耦

​			框架的使用人员：框架整合

​	3.分层架构下的常见框架

​			web:strust2,springmvc

​			service:spring

​			dao:mybatis,hibernate

## 知识点-MyBatis框架概述

### 1.目标

- [ ] 能够了解什么是MyBatis

### 2.路径

1. jdbc程序回顾
2. mybatis进行概述讲解

### 3.讲解

#### 3.1jdbc 程序回顾

##### 3.1.1程序回顾

​	1.加载驱动

​	2.获取连接

​	3.创建sql预处理prepareStatement

​	4.执行返回结果

​	5.遍历结果封装

​	6.关闭连接，释放资源

​	7.返回

##### 3.1.2 jdbc 问题分析 

1. 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。
2. Sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大， sql 变动需要改变java 代码。 
3. 使用 preparedStatement 向占有位符号传参数存在硬编码，因为 sql 语句的 where 条件不一定，可能多也可能少，修改 sql 还要修改代码，系统不易维护。  
4. 对结果集解析存在硬编码（查询列名）， sql 变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成 pojo 对象解析比较方便 

|      | jdbc | jdbctemplate | mybatis                     |
| ---- | ---- | ------------ | --------------------------- |
| 1    | 1    | 1            | 1                           |
| 2    |      |              | 1（sql映射文件）xml         |
| 3    |      |              | 1 (sql映射文件可以配置参数) |
| 4    |      | 1            | 1                           |

**JDBC代码**

![1558318496903](D:/mybatis/%E7%AC%AC1%E5%A4%A9/%E8%AE%B2%E4%B9%89/images/1558318496903.png)



**JdbcTemplate**

![1558318595942](D:/mybatis/%E7%AC%AC1%E5%A4%A9/%E8%AE%B2%E4%B9%89/images/1558318595942.png)

#### 3.2 MyBatis框架概述  

​		mybatis 是一个优秀的**基于 java 的持久层框架**，它同时也被称之为一个**半自动orm框架**，它内部**封装了 jdbc**，使开发者只需要关注 **sql 语句**本身，而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。

​	   mybatis 通过xml 或注解的方式将要执行的各种statement 配置起来，并通过java 对象和statement 中sql的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql并将结果映射为 java 对象并返回。

​	   采用 ORM 思想解决了实体和数据库映射的问题， 对jdbc 进行了封装，屏蔽了jdbc api 底层访问细节，使我们不用与 jdbc api打交道，就可以完成对数据库的持久化操作。

​		**orm(Object Relational Mapping)**:是一种为了解决面向对象与关系数据库存在的互不匹配的现象的技术思想。**简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系数据库中的一种思想**。

官网: http://www.mybatis.org/mybatis-3/

### 4.小结

1. 为什么要学习mybatis

   1.mybatis解决了jdbc和jdbctemplate没解决的问题

   2.国内很流行，工作肯定会用，所以要学。

  2.什么是mybatis

​		1.基于java开发

​		2.持久层框架

​		3.底层封装jdbc

​		4.半自动orm框架（javabean->sql映射文件->db表）

​		

# 第二章-Mybatis入门

## 案例-Mybatis快速入门  

### 1.需求

- [ ] 使用MyBatis查询所有的用户, 封装到List集合（select * from user）

### 2.步骤

​	准备环境

​			1.准备数据库

​			2.准备maven项目

​			3.引入pom依赖

​	mybatis准备工作

​			4.和数据库对应的实体类

​			5.sql映射文件

​			6.核心配置文件sqlMapConfig.xml

​	测试

​			7.log4j.properties

​			8.编写测试案例测试。

### 3.实现

#### 3.1准备工作

+ 数据库

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` datetime DEFAULT NULL COMMENT '生日',
  `sex` char(1) DEFAULT NULL COMMENT '性别',
  `address` varchar(256) DEFAULT NULL COMMENT '地址',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', '张三', '2019-06-21 15:47:34', '男', '北京');
INSERT INTO `user` VALUES ('2', '李四', '2019-06-30 15:24:58', '男', '西安');
INSERT INTO `user` VALUES ('3', '毛六', '2019-06-17 16:15:04', '男', '中山');
INSERT INTO `user` VALUES ('4', '王五', '2019-06-21 15:47:50', '男', '河北');
INSERT INTO `user` VALUES ('5', '赵六', '2019-06-21 15:48:12', '女', '山东');

DROP TABLE IF EXISTS `user_info`;
CREATE TABLE `user_info` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(32) NOT NULL COMMENT '用户名称',
  `user_birthday` datetime DEFAULT NULL COMMENT '生日',
  `user_sex` char(1) DEFAULT NULL COMMENT '性别',
  `user_address` varchar(256) DEFAULT NULL COMMENT '地址',
  PRIMARY KEY (`user_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user_info` VALUES ('1', '张三', '2019-06-21 15:47:34', '男', '北京');
INSERT INTO `user_info` VALUES ('2', '李四', '2019-06-30 15:24:58', '男', '西安');
INSERT INTO `user_info` VALUES ('3', '毛六', '2019-06-17 16:15:04', '男', '中山');
INSERT INTO `user_info` VALUES ('4', '王五', '2019-06-21 15:47:50', '男', '河北');
INSERT INTO `user_info` VALUES ('5', '赵六', '2019-06-21 15:48:12', '女', '山东');
```

#### 3.2.MyBatis快速入门

![1561619272823](D:\讲义\ssm框架\day01\笔记\img\1561619272823.png)



##### 3.2.1创建Maven工程(jar)导入坐标

```xml
         <dependencies>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>3.4.5</version>
            </dependency>
            <!--MySQL驱动包-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.6</version>
                <scope>runtime</scope>
            </dependency>

            <!--日志包-->
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>1.2.12</version>
            </dependency>

            <!--测试包-->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
                <scope>test</scope>
            </dependency>
			
    		<!--引入lombok简化java代码-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.16.10</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
```

##### 3.2.2创建User实体类

+ User .javal

```java
@Data- lombok插件的标记,用来代替一个类,类似于注释
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
} 
```

##### 3.2.3创建 UserMapper.xml 映射文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace属性: 接口类的全限定名--  唯一标识>
<mapper namespace="UserMapper">
    <!--select标签: 查询      ==>不同的标签代表不同的作用,标签名是sql语句的第一个字母
        id属性: 方法名        //通过这个方法可以直接执行方法里面的sql语句
        resultType属性: 写方法返回值类型(如果是list,直接写实体类的全限定名)===>定义返回值
        标签体: sql语句
    -->
    <select id="findAll" resultType="com.itheima.entity.User">
         SELECT * FROM  user
    </select>
</mapper>
```

##### 3.2.4创建 SqlMapConfig.xml 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--配置连接数据库的环境 default:指定使用哪一个环境-->
    <environments default="development">
        <environment id="development">
            <!--配置事务,MyBatis事务用的是jdbc-->
            <transactionManager type="JDBC"/>
            <!--配置连接池, POOLED:使用连接池(mybatis内置的); UNPOOLED:不使用连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3307/mybatis?characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!--引入映射文件; resource属性: 映射文件的路径-->
        <mapper resource="com/itheima/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

##### 3.2.5 创建log4j.properties

为了方便查看日志，在main/resources下创建log4j.properties文件，代码如下：

```properties
log4j.rootLogger=DEBUG,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
log4j.logger.org.apache=INFO
```



##### 3.2.6测试

```java
public class MybatisTest {
    @Test
    public void testFindAll() throws Exception {
        //1. 读取SqlMapConfig.xml获得输入流
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        //3. 获得SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //4.通过namespace.id调用
        List<User> users = sqlSession.selectList("UserMapper.findAll");
        //5.打印输出
       	for (User user : users) {
            System.out.println(user);
        }
        //6.释放资源
        sqlSession.close();
    }
}
```

### 4.小结

#### 4.1步骤

​	准备环境

​			1.准备数据库

​			2.准备maven项目

​			3.引入pom依赖

​	mybatis开发准备

​			1.javabean和数据库表对应

​			2.sql映射文件

​			3.核心配置文件SqlMapConfig.xml

​	测试准备工作

​			1.引入log4j.properties(方便查看执行过程中产生的sql)

​	测试

​			1.编写测试案例

## 案例二-Mapper动态代理方式实现dao开发

### 1.目标

- [ ] 通过Mapper动态代理方式实现dao开发

### 2.路径

1.改造入门案例

2.代理方式的规范

### 3.讲解

#### 3.1入门案例改造

##### 3.1.1 步骤

​	1.创建接口对应sql映射文件(方法名和crud的id对应，接口的全限定名和namespace对应，返回类型和resultType对应)

​	2.通过sqlSession.getMapper()获取接口的代理类

##### 3.1.2 dao接口

```java
public interface UserDao {
    public List<User> findAll();  //这里的返回值,要根据userMapper.xml文件sql语句对象的结果来判断
}
```

##### 3.1.3 xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace指向接口的全限定名-->
<mapper namespace="com.itheima.dao.UserDao">
    <select id="findAll" resultType="com.itheima.entity.User">
        SELECT * FROM user
    </select>
</mapper>
```

##### 3.1测试

```java
public class MybatisTest {

    @Test
    public void testFindAll() throws Exception {
        //1. 读取SqlMapConfig.xml获得输入流
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        //3.获得SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //4.获取接口的代理类
        UserDao mapperProxy = sqlSession.getMapper(UserDao.class);//动态代理和普通入门的差别在这里,用userDao.class代替了UserMapper.findAll,更加的降低了耦合性,这样的话就可以不用更改这里,而是直接更改接口类所连接的映射文件对象
        //5.调用代理方法
        List<User> users = mapperProxy.findAll();
        //6.打印输出
       	for (User user : users) {
            System.out.println(user);
        }
        //7.释放资源
        sqlSession.close();
    }

}
```

#### 3.2代理方式规范

​	1.sql映射文件的namespace要和接口的全限定名一致

​	2.sql映射文件中的sql节点的id要和接口的方法名一致

​	3.sql映射文件中的resulttype的id要和接口的返回类型一致

#### 3.3java代码里面设计模式

![1561017283666](img/1561017283666.png)

### 4.小结

​	1.使用到的设计模式

![image-20191228155402462](D:\讲义\ssm框架\day01\画图\image-20191228155402462.png)

​	2.核心要素

​		1.javabean和数据库对应

​		2.sql映射文件

​		3.核心配置文件

​		4.代理模式需要接口对映射文件做抽象封装

​	3.注意事项

​		1.sql映射文件

````xml
<!--
namespace：映射文件的唯一标识（当使用接口代理方式的时候要和接口的全限定名一致）
-->
<mapper namespace="com.itheima.dao.UserMapper">
    <!--
        id:crud节点的唯一标识（当使用接口代理方式的时候要和接口的方法名一致）
        resultType:要封装的实体类的全限定名(和接口的返回类型一致)
    -->
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>
````

​		2.核心配置文件

````xml
<configuration>
    <!--配置连接数据库的环境 default:指定使用哪一个环境-->
    <environments default="development">
        <environment id="development">
            <!--配置事务,MyBatis事务用的是jdbc-->
            <transactionManager type="JDBC"/>
            <!--配置连接池, POOLED:使用连接池(mybatis内置的); UNPOOLED:不使用连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3307/mybatis_84?characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
        <environment id="development1">
            <!--配置事务,MyBatis事务用的是jdbc-->
            <transactionManager type="JDBC"/>
            <!--配置连接池, POOLED:使用连接池(mybatis内置的); UNPOOLED:不使用连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3307/mybatis?characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!--引入映射文件; resource属性: 映射文件的相对路径-->
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
````

​		3.实体类

````java
@Data//使用它需要下载lombok插件，导入依赖
public class User {
    //字段名称和类型和数据库一致
    private Integer id;//int(11)
    private String username;//varchar(32
    private Date birthday;// datetime',
    private String sex; //char(1)
    private String address;//varchar(256)
}
````



## 知识点-核心配置文件详解

### 1.目标

- [ ] 掌握SqlMapConfig.xml配置文件

### 2.路径

​	1.掌握核心配置文件的配置顺序

​	2.掌握properties配置方式

​	3.掌握typeAliases配置

​	4.掌握mappers配置

### 3.讲解

#### 3.1.核心配置文件的顺序

**properties（引入外部properties文件）**

settings（全局配置参数）

**typeAliases（类型别名）**

typeHandlers（类型处理器）

objectFactory（对象工厂）

plugins（插件）

**environments（环境集合属性对象）**

​	environment（环境子属性对象）

​		transactionManager（事务管理）

​		dataSource（数据源）

**mappers（映射器）**

#### 3.2.properties

步骤:

1. 抽取四个基本项到jdbc.properties
2. 在SqlMapConfig.xml使用properties标签引入jdbc.properties
3. 动态获得赋值



- jdbc.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3307/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=123456
```

- 引入到核心配置文件

```xml
<configuration>
   <properties resource="jdbc.properties">
    </properties>
    <!--数据源配置-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="UNPOOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.user}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    ....
</configuration>
```

#### 3.3.typeAliases（类型别名）

##### 3.3.1定义单个别名【了解】

- 核心配置文件

```xml
<typeAliases>
      <typeAlias type="com.itheima.entity.User" alias="asdasd"/>
 </typeAliases>
```

- 修改UserDao.xml

```java
<select id="findAll" resultType="asdasd">
    SELECT  * FROM  user
</select>
```

##### 3.3.2批量定义别名【重点】

使用package定义的别名：就是pojo的类名，大小写都可以

- 核心配置文件

```xml
<typeAliases>
    <package name="com.itheima.entity"/>
</typeAliases>
```

- 修改UserDao.xml

```java
<select id="findAll" resultType="user">
         SELECT  * FROM  user
</select>
```

#### 3.4.Mapper

##### 3.4.1方式一:引入映射文件路径

```xml
<mappers>
     <mapper resource="com/itheima/dao/UserDao.xml"/>
 </mappers>
```

##### 3.4.2方式二:扫描接口

- 配置单个接口

```xml
<mappers>
 	<mapper class="com.itheima.dao.UserDao"></mapper>
</mappers>
```

- 批量配置

```xml
<mappers>
   <package name="com.itheima.dao"></package>
</mappers>
```

### 4.小结

1. 顺序 

   ​	核心配置文件中configuration节点的顺序

   ​	properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?

2. properties（属性） 

   配置

   ```xml
   <!--
           resource：配置文件的相对路径
           属性会被存到map里面，相同key键会覆盖
           配置文件覆盖property属性，通过${key}取值
       -->
       <properties resource="jdbc.properties">
           <!--配置单的属性的kv-->
           <property name="jdbc.driver" value="com.mysql.jdbc.Driver"/>
       </properties>
   
   ```
   
   引用
   
   ```xml
<dataSource type="POOLED">
       <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
       <property name="username" value="${jdbc.username}"/>
       <property name="password" value="${jdbc.password}"/>
   </dataSource>
   ```
   
3. typeAliases

   ```xml
   别名不区分大小写 
   <typeAliases>
           <!--
               typeAlias：配置单个别名
                   type；类的全限定名
                   alias:别名
           -->
           <!--<typeAlias type="com.itheima.domain.User" alias="asd"/>-->
           <!--
               package：包扫描方式
                   把别名配置成类名
           -->
           <package name="com.itheima.domain"/>
       </typeAliases>
   ```
   
4. mappers

   ```xml
   <mappers>
           <!--1.扫描sql映射文件的位置-->
           <!--引入sql映射文件,resource填mapper文件的相对路径-->
          <!-- <mapper resource="mappers/UserMapper.xml"/>-->
   
   
           <!--2.通过接口扫描
               约定大于配置  1.接口名称要和sql映射文件名称一致
                           2.接口路径要和sql映射文件路径一致
           -->
           <!--<mapper class="com.itheima.dao.UserDao"/>-->
           <!--
               package:包扫描的方式
                   name；接口所在的包路径
           -->
           <package name="com.itheima.dao"/>
       </mappers>
   ```

# 第三章-MyBatis进阶

## 案例-使用Mybatis完成CRUD（增删改查）

### 1.需求

- [ ] 使用Mybatis完成CRUD

### 2.步骤

1.改造测试类，方便测试

2.在接口中创建对应的crud方法

3.在xml映射文件中实现对应的crud的sql节点

4.写测试方法进行测试

### 3.实现

#### 3.1新增用户

##### 3.1.1实现步骤

- UserDao中添加新增方法 

```java
public interface UserDao {
    /**
     * 保存用户
     * @param user
     */
    void save(User user);
}
```

- 在 UserDao.xml 文件中加入新增配置 

```xml
    <insert id="save" parameterType="com.itheima.bean.User">
        INSERT INTO t_user(username,sex,birthday,address) VALUES (#{username},#{sex},#{birthday},#{address})
    </insert>

	<!--我们可以发现， 这个 sql 语句中使用#{}字符， #{}代表占位符，我们可以理解是原来 jdbc 部分所学的?，它们都是代表占位符， 具体的值是由 User 类的 username 属性来决定的。
	parameterType 属性：代表参数的类型，因为我们要传入的是一个类的对象，所以类型就写类的
全名称。-->
```

- 添加测试类中的测试方法 

```java
    @Test
    public void fun02() throws Exception {
        //1. 读取SqlMapConfig.xml获得输入流
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        //3. 获得SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //4.获得UserDao代理对象
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        //5.调用方法
        User user = new User("赵六","男",new Date(),"广州");
        userDao.save(user);
        sqlSession.commit();

        //6.释放资源
        sqlSession.close();
    }
```

##### 3.1.2新增用户 id 的返回值 

​	新增用户后， 同时还要返回当前新增用户的 id 值，因为 id 是由数据库的自动增长来实现的，所以就相当于我们要在新增后将自动增长 auto_increment 的值返回。 

- 方式一 SelectKey获取主键

| **属性**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| keyProperty | selectKey 语句结果应该被设置的目标属性。                     |
| resultType  | 结果的类型。MyBatis 通常可以算出来,但是写上也没有问题。MyBatis 允许任何简单类型用作主键的类型,包括字符串。 |
| order       | 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE,那么它会首先选择主键,设置 keyProperty 然后执行插入语句。如果设置为 AFTER,那么先执行插入语句,然后是 selectKey 元素-这和如  Oracle 数据库相似,可以在插入语句中嵌入序列调用。 |

  UserDao.xml

```xml
    <!--parameterType属性: 参数的类型 ;  赋值的时候直接写对象里面的属性名-->
    <insert id="save" parameterType="com.itheima.bean.User">
        <!--presultType: 主键类型; keyProperty:pojo里面对应的id的属性名; order属性: 指定是在目标的sql语句之前还是之后执行 -->
        <selectKey resultType="int" keyProperty="uid" order="AFTER">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO t_user(username,sex,birthday,address)VALUES(#{username},#{sex},#{birthday},#{address})
    </insert>
```

- 方式二属性配置

  UserDao.xml

```xml
<insert id="save" parameterType="com.itheima.bean.User" keyProperty="uid" useGeneratedKeys="true">
        INSERT INTO t_user(username,sex,birthday,address) VALUES (#{username},#{sex},#{birthday},#{address})
    </insert>
```

- 方式三 全局配置

  SqlMapConfig.xml

```xml
<settings>
    <setting name="useGeneratedKeys" value="true"/>
</settings>
```

​	  UserDao.xml

````xml
<insert id="save" parameterType="com.itheima.bean.User" keyProperty="uid">
        INSERT INTO t_user(username,sex,birthday,address) VALUES (#{username},#{sex},#{birthday},#{address})
    </insert>
````



##### 3.1.3新增用户 id 的返回值(字符串类型)

```xml
<insert id="save" parameterType="com.itheima.bean.User">
    <selectKey  keyProperty="id" resultType="string" order="BEFORE">
        select uuid()
    </selectKey>
        INSERT INTO t_user(username,sex,birthday,address) VALUES (#{username},#{sex},#{birthday},#{address})
</insert>
```

#### 3.2修改用户

- UserDao中添加新增方法 

```java
public interface UserDao {
    /**
     * 更新用户
     * @param user
     */
    void  update(User user);
}
```

- 在 UserDao.xml 文件中加入新增配置 

```xml
<update id="update" parameterType="com.itheima.bean.User">
    UPDATE t_user SET username=#{username},sex=#{sex},birthday=#{birthday},address=#{address} WHERE uid=#{uid}
</update>
```

- 添加测试类中的测试方法 

```java
    @Test
    public void fun03() throws Exception {
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        User user = new User("王二","女",new Date(),"广州");
        user.setUid(11);
        userDao.update(user);

        sqlSession.commit();
        sqlSession.close();
    }
```

#### 3.3删除用户

- UserDao中添加新增方法 

```java
public interface UserDao {

    /**
     * 删除用户
     * @param uid
     */
    void delete(int uid);
}
```

- 在 UserDao.xml 文件中加入新增配置 

```xml
<delete id="delete" parameterType="int">
    DELETE FROM t_user WHERE uid = #{uid}
</delete>
```

- 添加测试类中的测试方法 

```java
    @Test
    public void fun04() throws Exception {
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        userDao.delete(11);

        sqlSession.commit();
        sqlSession.close();
    }
```

#### 3.4模糊查询

##### 3.4.1 方式一

- UserDao 中添加新增方法 

```java
public interface UserDao {
    /**
     * 模糊查询
     * @param firstName
     * @return
     */
    List<User> findByFirstName(String firstName);
}
```

- 在 UserDao.xml 文件中加入新增配置 

```xml
<select id="findByFirstName" parameterType="string" resultType="com.itheima.bean.User">
  	SELECT * FROM t_user WHERE username LIKE concat('%',#{firstName},'%')
</select>
```

- 添加测试类中的测试方法 

```java
    @Test
    public void fun05() throws Exception {
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        List<User> list = userDao.findByFirstName("王");
        System.out.println(list);

        sqlSession.commit();
        sqlSession.close();
    }
```

##### 3.4.2 方式二

- UserDao 中添加新增方法 

```java
public interface UserDao {
    /**
     * 模糊查询
     * @param firstName
     * @return
     */
    List<User> findByFirstName02(String firstName);
}
```

- 在 UserMapper.xml 文件中加入新增配置 

```xml
    <select id="findByFirstName02" parameterType="string" resultType="com.itheima.bean.User">
        SELECT * FROM t_user WHERE username LIKE '%${value}%'
    </select>
```

​	` 我们在上面将原来的#{}占位符，改成了${value}。注意如果用模糊查询的这种写法，那么${value}的写法就是固定的，不能写成其它名字。` 

- 添加测试类中的测试方法 

```java
    @Test
    public void fun06() throws Exception {
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = builder.build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        List<User> list = userDao.findByFirstName02("王");
        System.out.println(list);

        sqlSession.commit();
        sqlSession.close();
    }
```

##### 3.4.3 #{}与${}的区别【面试】

1. `#{}`表示一个占位符号  
   + 通过`#{}`可以实现 preparedStatement 向占位符中设置值,自动进行 java 类型和 数据库 类型转换 
   +  `#{}`可以有效防止 sql 注入 
   +  `#{}`可以接收简单类型值(基本类型,String)或 pojo 属性值
   +   如果 parameterType 传输单个简单类型值(基本类型,String)， `#{}` 括号中可以是任意名称。
2. `${}`表示拼接 sql 串
   + 通过`${}`可以将 parameterType 传入的内容拼接在 sql 中且不进行 jdbc 类型转换. 
   + `${}`不能防止 sql 注入
   + `${}`可以接收简单类型值或 pojo 属性值
   + ==如果 parameterType 传输单个简单类型值.`${}`括号中只能是 value== 

#### 3.5.SqlSessionFactory工具类的抽取

```java
public class SqlSessionFactoryUtils {

    public static SqlSessionFactory sqlSessionFactory;

    static{
        InputStream is = null;
        try {
            //读取SqlMapConfig
            is = Resources.getResourceAsStream("SqlMapConfig.xml");
            //构建sqlSessionFactory
            SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
            sqlSessionFactory = builder.build(is);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
    /**
     * 获得Session
     * @return
     */
    public static SqlSession openSession(){
        SqlSession sqlSession = sqlSessionFactory.openSession();
        return  sqlSession;
    }
}
```

### 4.小结

#### 4.1CRUD

​	1.新增

​		<insert>

​		返回key键的三种方式

​		方式一：selectkey

````xml
<!--
            返回主键方式一：
            selectKey查询主键
                   keyProperty：查询出来的主键存储到user的哪个属性
                   resultType：返回类型
                   order:当前的selectKey执行时机
                        BEFORE代表在插入数据之前执行
                        AFTER:在插入数据之后执行

        -->
       <selectKey keyProperty="id" resultType="int" order="AFTER">
            select last_insert_id()
        </selectKey>
````

​		方式二: useGeneratedKeys="true" keyProperty="id"

​		方式三：可以将方式二的useGeneratedKeys="true"移到核心配置文件中

````xml
 <settings>
        <!--useGeneratedKeys="true"-->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>
````

​		xml

```xml
<!--
    parameterType:参数类型，对传入的参数类型进行限制
    insert into user(username, birthday, sex, address)
    values(?,?,?,?)
    #{}:固定写法，表示一个占位符
    当传入参数为javabean时或者为map时，解析#{}填对应的属性名或key键
-->
<insert id="addUser" parameterType="User" keyProperty="id">
<!--
    keyColumn:数据中主键对应的列,可以不写
    keyProperty:javabean中主键对应的属性
    resultType:返回的数据要转化的javaType
    order:
            AFTER:在执行insert语句之后执行selectkey中的sql
            BEFORE:在执行insert语句之前执行selectkey中的sql
-->
   <!--<selectKey keyProperty="id" resultType="int" order="AFTER">
       select last_insert_id()
   </selectKey>-->
   insert into user(username, birthday, sex, address)
   values(#{username}, #{birthday}, #{sex}, #{address})
</insert>
```

2.查询

```xml
<!--
    使用MyBatis查询所有的用户, 封装到List集合
    id:xml文件当中sql节点的唯一标识，代理方式下必须和接口中方法名一致
    resultType:将结果集转化为它指定的javabean对象
-->
<select id="findAll" resultType="uSer">
    select * from user
</select>
```

3.删除

```xml
<!--
   如果参数为简单数据类型#{}方式取值时，里面可以随便写，
   #{}方式取值可以自动进行类型转换（javaType到jdbcType）
-->
    <delete id="deleteByid" parameterType="int">
        delete from user where id=#{asd}
    </delete>
```

4.修改

```xml
<update id="updateUser">
   update user set
       username=#{username},
       birthday=#{birthday},
       sex=#{sex},
       address=#{address}
   where id=#{id}
</update>
```

```XML
<select id="findAll" resultType="uSer">
    select * from user
</select>
```

模糊查询

​	简单数据类型

​	方式一： concat('%',#{随便写},'%')

​	方式二：'%${value}%'

面试

![image-20191220140115229](img\image-20191220140115229.png)

#### 4.2注意事项

+ 通过 `${}` 获得参数的值, 参数类型是简单类型(字符串,基本类型), 必须写 `${value}` 



## 知识点-parameterType深入

### 1.目标

- [ ] 掌握Mybatis的参数深入(parameterType)

### 2.路径

1.简单数据类型 可以

2.pojo类型 可以

3.传入复杂的javabean 可以

### 3.讲解

parameterType代表的含义是限定了传入的参数类型

#### 3.1传递简单类型

​	基本的类型,字符串

​	直接写#{任意内容}或者${value}

 ![1535099951604](img/1535099951604.png)

#### 3.2传递 pojo 对象 

​	Mybatis 使用 ognl 表达式解析对象字段的值， #{}或者${}括号中的值为 pojo 属性名称。 

![1535100050378](img/1535100050378.png)

#### 3.3传递 pojo 包装对象类型 

​	开发中通过 pojo 传递查询条件 ，查询条件是==综合的查询条件==，不仅包括用户查询条件还包括其它的查询条件（比如将用户购买商品信息也作为查询条件），这时可以使用包装对象传递输入参数。Pojo 类中包含 pojo。

​	京东查询的例子:

![1539830569011](img/1539830569011.png)

​	需求：根据用户id查询用户信息，查询条件放到 QueryVo 的 user 属性中。 

- QueryVo  

```java
public class QueryVo {

    private User user;

    public void setUser(User user) {
        this.user = user;
    }

    public User getUser() {
        return user;
    }
}
```

- UserDao接口 

```java
public interface UserDao {
    /**
     * 复杂参数查询
     * @return
     */
    List<User> findByQueryVo(QueryVo queryVo);

}
```

- UserDao.xml文件 

```xml
<mapper namespace="com.itheima.dao.UserDao">
    <select id="findByQueryVo" resultType="com.itheima.bean.User" parameterType="com.itheima.bean.QueryVo">
        SELECT * FROM t_user WHERE uid > #{user.uid}
    </select>
</mapper>
```

- 测试代码

```java
   @Test
    public void fun01() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtils.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        QueryVo queryVo = new QueryVo();
        User user = new User();
        user.setUid(2);
        queryVo.setUser(user);
        List<User> list = userDao.findByQueryVo(queryVo);
        System.out.println(list);

        sqlSession.close();
    }
```

### 4.小结

1.传入简单类型 #{任意名称} ${必须写value}

2.传入pojo类，解析的时候，解析属性 #{属性名} ${属性名}

3.复杂的pojo类,解析 #{对象属性.属性名} ${对象属性.属性名}

## 知识点-resultType深入   

### 1.目标

- [ ] 掌握Mybatis的参数深入(resultType)

### 2.路径

1.返回简单数据类型

2.返回pojo类

3.返回pojo类列表

4.resultMap讲解

### 3.讲解

#### 3.1输出简单类型

​	直接写对应的Java类型. eg: 返回int

```xml
<select id="findCount" parameterType="int" resultType="int">
     SELECT  COUNT(*) FROM t_user
</select>
```

#### 3.2输出pojo对象

​	直接写当前pojo类的全限定名 eg: 返回User

```xml
<select id="findByUid" parameterType="int" resultType="com.itheima.bean.User">
        select * from t_user where uid = #{uid}
</select>
```

#### 3.3输出pojo列表

​	直接写当前pojo类的全限定名 eg: 返回 List<User> list;

```xml
<select id="findAll" resultType="com.itheima.bean.User">
        select * from t_user
</select>
```

#### 3.4 resultMap结果类型 

       resultType可以指定pojo将查询结果映射为pojo，但需要pojo的属性名和sql查询的列名一致方可映射成功。

       如果sql查询列名和pojo的属性名不一致，可以通过resultMap将字段名和属性名作一个对应关系 ，resultMap实质上还需要将查询结果映射到pojo对象中。

    	resultMap可以实现将查询结果映射为复杂类型的pojo，比如在查询结果映射对象中包括pojo和list实现一对一查询和一对多查询。(下次课讲)

​	那我们今天就来看返回的列名与实体类的属性不一致时的情况. 下次课再接着研究复杂的封装(多表查询)

​	通过改别名的方式，现在返回结果集的列名已经与 User 类的属性名不相同了。 

```sql
select uid uid_,username username_ ,birthday birthday_ ,sex sex_ ,address address_  from t_user where uid = #{id}
```

- UserDao.java

```java
public interface UserDao {
    /**
     * 根据uid查询
     * @param uid
     * @return
     */
    User findByUid(int uid);

}
```

- UserDao.xml

```xml
    <select id="findByUid" parameterType="int" resultMap="findByUidMap">
        select uid uid_,username username_ ,birthday birthday_ ,sex sex_ ,address address_  from t_user where uid = #{id}
    </select>

    <resultMap id="findByUidMap" type="com.itheima.bean.User">
        <id property="uid" column="uid_"></id>
        <result property="username" column="username_"></result>
        <result property="birthday" column="birthday_"></result>
        <result property="sex" column="sex_"></result>
        <result property="address" column="address_"></result>
    </resultMap>
<!-- 
id：此属性表示查询结果集的唯一标识，非常重要。如果是多个字段为复合唯一约束则定义多个id。
type: 当前resultMap封装后返回的结果
property：表示 User 类的属性。
column：表示 sql 查询出来的字段名。(column 和 property 放在一块儿表示将 sql 查询出来的字段映射到指定的 pojo 类属性上。)
-->
```

### 4.小结

1.返回类型为简单类型 可以

2.返回类型为javabean 可以

3.返回类型和数据库字段不匹配

​	方案一:别名方式解决

```xml
 <!--
        解决数据库字段和属性名不一致问题
            方式一：使用别名
         user_id, user_name, user_birthday, user_sex, user_address
    -->
    <select id="findAll" resultType="UserInfo">
        select
         user_id userId,
         user_name userName,
         user_birthday userBirthday,
         user_sex userSex,
         user_address userAddress
        from user_info
    </select>
```

​	方案二：通过自定义resultmap解决

```xml
 	<resultMap id="userResult" type="UserInfo">
        <!--
            方式二：使用resultMap
        -->
        <!--
            id：标识当前的字段为主键
               column：查询出来的列名
               property：对应type的属性名
        -->
        <id column="user_id" property="userId"/>
        <result column="user_name" property="userName"/>
        <result column="user_birthday" property="userBirthday"/>
        <result column="user_sex" property="userSex"/>
        <result column="user_address" property="userAddress"/>
    </resultMap>


    <select id="findAll2" resultMap="userResult">
        select
         user_id userId,
         user_name userName,
         user_birthday userBirthday,
         user_sex userSex,
         user_address userAddress
        from user_info
    </select>
```

方案三：使用配置文件解决（只适用于下划线转驼峰）

```xml
<!--
    mapUnderscoreToCamelCase:下划线转驼峰
-->
<setting name="mapUnderscoreToCamelCase" value="true"/>
```

