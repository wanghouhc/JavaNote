

# SpringMVC第3天笔记

# 第1章 **课程计划**。

 

 今日学习目标：

掌握ssm整合（SpringMVC+spring+mybatis）（第2章） 

# 第2章 SSM整合 

## 2.1 **搭建整合环境**

### 【目标】

学会使用SSM框架进行开发，完成从页面到数据库的新增、查询功能

### 【路径】

（1）开发准备

第一步：创建web工程

第二步：导入jar包

第三步：创建实体类

第四步：创建Dao

第五步：创建Service

第六步：创建Controller

（2）搭建Spring的环境

第一步：applicationContext.xml

第二步：log4j.properties文件

第三步：AccountService.java

第四步：测试类

（3）搭建SpringMVC环境

第一步：web.xml

第二步：springmvc.xml

第三步：AccountController.java

第四步：index.jsp

第五步：success.jsp（WEB-INF/pages/success.jsp）

（4）Spring整合SpringMVC框架

第一步：web.xml

第二步：AccountController.java（注入Service）

（5）Mybatis框架

第一步：sqlMapConfig.xml

第二步：AccountDao.java（AccountDao.xml）

第三步：测试TestMybatis.java

（6）Spring整合Mybatis框架

第一步：applicationContext.xml（重点）

第二步：AccountDao.java

第三步：TestSpringMybatis.java

第四步：AccountServiceImpl.java

第五步：AccountController.java

第六步：success.jsp

（7）配置Spring的声明式事务处理

第一步：applicationContext.xml

第二步：index.jsp（新增表单）

第三步：AccountController.java（保存）

### 【讲解】

1：整合说明：SSM整合可以使用多种方式，咱们会选择==XML（第三方对象） + 注解（自己new的对象）==的方式

![img](./img/001.png) 

2：整合的思路

 

（1）：先搭建整合的环境

 

（2）：先把Spring的配置搭建完成

 

（3）：再使用Spring整合SpringMVC框架

 

（4）：最后使用Spring整合MyBatis框架

 

3：创建数据库和表结构

语句

```sql
create database itcastspringmvc;
use itcastspringmvc;
create table account(
    id int primary key auto_increment,
    name varchar(20),
    money double
);
```

 

4：创建maven的工程

 

## 2.2 **开发准备**

### 2.2.1 **第一步：创建web工程**

项目名：springmvc_day03_ssm

![img](./img/003.png) 

 

### 2.2.2 **第二步：导入jar包**

```xml
<packaging>war</packaging>
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <mysql.version>5.1.6</mysql.version>
    <mybatis.version>3.4.5</mybatis.version>
</properties>

<dependencies>

    <!-- spring -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.6.8</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <!-- log start -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <!-- log end -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.0</version>
    </dependency>

    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
        <type>jar</type>
        <scope>compile</scope>
    </dependency>

</dependencies>

<build>
    <finalName>ssm</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <showWarnings>true</showWarnings>
                </configuration>
            </plugin>
        </plugins>
</build>
```

 

### 2.2.3 **第三步：创建实体类**

创建包com.itheima.ssm.domain，创建类Account.java

```java
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Double money;
    // 提供set和get方法
}
```

 

### 2.2.4 **第四步：创建Dao**

创建包com.itheima.ssm.dao，创建类AccountDao.java

【路径】

查询所有findAll方法

新增saveAccount方法

```java
/**
 * AccountMapper接口，不用编写实现类，框架生成代理对象
 */
public interface AccountDao {

    public List<Account> findAll();

    public void saveAccount(Account account);

}
```

 

### 2.2.5 **第五步：创建Service**

创建包com.itheima.ssm.service，创建接口AccountService.java

```java
public interface AccountService {

    public List<Account> findAll();

    public void saveAccount(Account account);


}
```

 

创建类AccountServiceImpl，实现接口

```java
/**
 * 账号的实现类
 */
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    public List<Account> findAll() {
        System.out.println("业务层：查询所有的帐户...");
        return accountDao.findAll();
    }

    public void saveAccount(Account account) {
        System.out.println("业务层：保存帐户...");
        accountDao.saveAccount(account);
    }

}
```

 

### 2.2.6 **第六步：创建Controller**

创建包com.itheima.ssm.controller，创建类AccountController.java

```java
/**
 * 帐户
 */
public class AccountController {

    private AccountService accountService;

    /**
     * 查询所有
     * @return
     */
    public String findAll() {
        System.out.println("表现层：查询所有账户...");
        List<Account> list = accountService.findAll();
        for (Account account : list) {
            System.out.println(account);
        }
        return "success";
    }

    /**
     * 保存
     * @return
     */
    public String saveAccount(Account account) {
		System.out.println("表现层：保存账户...");
        accountService.saveAccount(account);
        return "success";
    }

}
```

 

最终包和类的结构是：

![img](./img/005.png) 

【小结】：先写各层的代码，完成框架的准备工作

## 2.3 **搭建Spring的环境**

### 2.3.1 **applicationContext.xml**

在项目中的resources文件夹下创建applicationContext.xml的配置文件，编写具体的配置信息。

【路径 】：配置组件扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop.xsd
   http://www.springframework.org/schema/tx 
   http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.itheima.ssm">
        <!-- 忽略某些注解，把Controller给忽略了 -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>
```

 

### 2.3.2 **log4j.properties文件**

 

```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=info, CONSOLE, LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=d:/axis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
```



### 2.3.3 **AccountService.java**

添加@Service("accountService")

```java
/**
 * 账号的实现类
 */
@Service("accountService")
public class AccountServiceImpl implements AccountService {

}
```



### 2.3.4 **测试类**

在项目中编写测试方法，进行测试

创建测试类：com.itheima.test.TestService.java

```java
public class TestService {

    /**
     * 测试Service层
     */
    @Test
    public void testService(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        AccountService as = (AccountService) ac.getBean("accountService");
        as.findAll();
    }
}
```

测试结果：

![img](./img/006.png) 

## 2.4 **SpringMVC框架**

### 2.4.1 **web.xml**

1：在web.xml中配置DispatcherServlet前端控制器

```xml
<!-- 配置前端控制器：服务器启动必须加载，需要加载springmvc.xml配置文件 -->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 配置初始化参数，创建完DispatcherServlet对象，加载springmvc.xml配置文件 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!-- 服务器启动的时候，让DispatcherServlet对象创建 -->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```



2：在web.xml中配置CharacterEncodingFilter过滤器解决post请求中文乱码

```xml
<!-- 配置解决中文乱码的过滤器 -->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

 

### 2.4.2 **springmvc.xml**

在resources下，创建springmvc.xml

创建springmvc.xml的配置文件

【路径】

1：组件扫描配置，扫描controller的注解，排除Service注解的扫描

2：配置视图解析器 

3：设置静态资源不过滤，可以使用

4：开启对SpringMVC注解的支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 1：组件扫描配置，扫描controller的注解，排除Service注解的扫描 -->
    <context:component-scan base-package="com.itheima.ssm">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <!-- 排除Service注解的扫描，让spring扫描，否则事务可能会失效 -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"></context:exclude-filter>

    </context:component-scan>

    <!-- 2:配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- JSP文件所在的目录 -->
        <property name="prefix" value="/WEB-INF/pages/" />
        <!-- 文件的后缀名 -->
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- 3:设置静态资源不过滤，可以使用：-->
<mvc:default-servlet-handler></mvc:default-servlet-handler>

<!-- 
     <mvc:resources location="/css/" mapping="/css/**" />
    <mvc:resources location="/images/" mapping="/images/**" />
    <mvc:resources location="/js/" mapping="/js/**" />
-->


    <!-- 4:开启对SpringMVC注解的支持 -->
    <mvc:annotation-driven />

</beans>
```

其中：

<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>

表示springmvc的注解包含@Controller的注解。

为什么要包含@Controller注解呢？

1）SpringMVC IOC容器  存在Controller的实例对象

2）Spring IOC容器  存在Controller的实例和Service层实例以及Dao层实例

 

==组件扫描重叠会引发一些问题：==

==例如：事务失效、相关注解失效==

​	解决方案一：

​		SpringMVC IOC容器(springmvc.xml）：只扫描 @Controller 注解、不扫描@Service注解

​		Spring IOC容器（applicationContext.xml）：不扫描 @Controller 注解

​	==解决方案二：（建议使用）==

==只让SpringMVC扫描，不让Spring扫描==

​        **在springMVC的配置中添加<import resource="applicationContext.xml"></import>**

### 2.4.3 **AccountController.java**

```java
/**
 * 帐户
 */
@Controller
@RequestMapping(path = "/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    /**
     * 查询所有
     * @return
     */
    @RequestMapping(path = "/findAll")
    public String findAll() {
        System.out.println("表现层：查询所有账户...");
        return "success";
    }
}
```

 

### 2.4.4 **index.jsp**

```html
<body>
<h3>账号操作</h3>
<a href="account/findAll">查询所有账号</a>
</body>
```

 

### 2.4.5 **success.jsp（WEB-INF/pages/success.jsp）**

```html
<body>
<h1>访问成功</h1>
</body>
```

发布到tomcat，进行测试

![img](./img/007.png) 

  

## 2.5 **Spring整合SpringMVC框架**

目的：在controller中能成功的调用service对象中的方法。（由spring创建对象，注入到Controller中使用）

在项目启动的时候，就去加载applicationContext.xml的配置文件，在web.xml中配置ContextLoaderListener监听器（默认该监听器只能加载WEB-INF目录下的applicationContext.xml的配置文件）。使用contextConfigLocation的属性让它加载类路径下的applicationContext.xml。

### 2.5.1 **web.xml**

目的：web容器启动，加载spring容器

```html
<!-- 配置Spring的监听器，默认加载WEB-INF下的applicationContext.xml -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!-- 该监听器默认只加载WEB-INF下的applicationContext.xml配置文件，通过contextConfigLocation改变位置 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

 

### 2.5.2 **AccountController.java**

添加： @Autowired

```java
@Controller
@RequestMapping(path = "/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    /**
     * 查询所有
     * @return
     */
    @RequestMapping(path = "/findAll")
    public String findAll() {
        System.out.println("表现层：查询所有账户...");
        List<Account> list = accountService.findAll();
        System.out.println(list);
        return "success";
    }
}
```

测试：

![img](./img/008.png) 

## 2.6 **Mybatis框架**

### 2.6.1 **sqlMapConfig.xml**

在web项目中编写==sqlMapConfig.xml==的配置文件，放置到resources下，编写核心配置文件

【路径】

1：连接数据库

2：加载配置文件/注解的映射

3：添加别名

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!--3：别名-->
    <typeAliases>
        <package name="com.itheima.ssm.domain"></package>
    </typeAliases>
  
    <!--1：连接数据库-->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///itcastspringmvc"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 2：加载配置文件/注解的映射（使用注解） -->
    <mappers>
        <!-- <mapper class="com.itheima.ssm.dao.AccountDao"/> -->
        <!-- 该包下所有的dao接口都可以使用 -->
        <package name="com.itheima.ssm.dao"/>
    </mappers>
</configuration>
```

Xml的回顾：我们采用的是注解，如果使用配置文件AccountDao.xml，可以进行使用如下配置

【路径】

1：select标签：查询所有

2：insert标签：新增

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.ssm.dao.AccountDao">
    <!--查询所有-->
    <select id="findAll" resultType="com.itheima.ssm.domain.Account">
        select * from account
    </select>
    <!--保存-->
    <insert id="save" parameterType="com.itheima.ssm.domain.Account">
        insert into account(name,money) values (#{name},#{money})
    </insert>
</mapper>
```



### 2.6.2 **AccountDao.java**

在AccountDao接口的方法上添加注解，编写SQL语句

```java
/**
 * AccountDao接口，不用编写实现类，框架生成代理对象
 */
public interface AccountDao {

    @Select(value = "select * from account")
    public List<Account> findAll();

    @Insert(value = "insert into account (name,money) values (#{name},#{money})")
    public void saveAccount(Account account);

}
```

Xml的回顾：我们采用的是注解==AccountDao.xml==



### 2.6.3 **测试TestMybatis.java**

在测试环境下，com.iteheima.test中，编写测试方法

```java
public class TestMybatis {
    @Test
    public void testFindAll() throws Exception {
        // 加载配置文件
        InputStream in = Resources.getResourceAsStream("sqlMapConfig.xml");
        // 创建工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
        // 创建sqlSession对象
        SqlSession sqlSession = sessionFactory.openSession();
        // 获取代理对象，调用Dao
        AccountDao accountDao = sqlSession.getMapper(AccountDao.class);
        // 获取查询所有的方法
        List<Account> list = accountDao.findAll();
        for(Account account:list){
            System.out.println(account);
        }
        // 释放资源
        sqlSession.close();
        in.close();
    }

    @Test
    public void testSave() throws Exception {
        Account account = new Account();
        account.setName("小菲");
        account.setMoney(400d);

        // 加载配置文件
        InputStream in = Resources.getResourceAsStream("sqlMapConfig.xml");
        // 创建工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
        // 创建sqlSession对象
        SqlSession sqlSession = sessionFactory.openSession();
        // 获取代理对象
        AccountDao accountDao = sqlSession.getMapper(AccountDao.class);

        accountDao.saveAccount(account);

        // 提交事务
        sqlSession.commit();
        // 释放资源
        sqlSession.close();
        in.close();
    }
}
```

 别忘记，如果是新增，需要：sqlSession.commit();

问题：在使用Mybatis的时候，测试的时候，需要创建SqlSessionFactory、SqlSession，才能调用Dao的接口，给开发带来了耦合，编码的麻烦，那怎么办呢？

## 2.7 Spring整合MyBatis框架

### 2.7.1 **applicationContext.xml（重点）**

目的：把SqlMapConfig.xml配置文件中的内容配置到applicationContext.xml配置文件中

【路径】

1：数据源配置

2：创建SqlSession工厂类（核心）

3：配置扫描dao的包，让Dao层的接口被spring管理

```xml
<!-- 1：配置C3P0的连接池对象 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///itcastspringmvc" />
    <property name="user" value="root" />
    <property name="password" value="root" />
</bean>

<!--2：配置SqlSession的工厂 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <!--添加别名-->
  	<property name="typeAliasesPackage" value="com.itheima.ssm.domain"></property>
  	<!--加载mybatis的配置文件-->
  	<!--<property name="configLocation">-->
  		<!--<value>classpath:sqlMapConfig.xml</value>-->
  	<!--</property>-->
  	<!--加载mybatis的映射文件-->
  	<!--<property name="mapperLocations">-->
  		<!--<array>-->
  			<!--<value>classpath:com/itheima/ssm/dao/*Dao.xml</value>-->
  		<!--</array>-->
  	<!--</property>-->
</bean>

<!--3：配置扫描dao的包，让Dao层的接口被spring管理 -->
<bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.itheima.ssm.dao"/>
</bean>
```

SqlMapConfig.xml配置文件可以删除。

### 2.7.2 **AccountDao.java**

在AccountDao接口中添加@Repository注解，也可以不填加。

```java
@Repository
public interface AccountDao {

    @Select(value = "select * from account")
    public List<Account> findAll();

    @Insert(value = "insert into account (name,money) values (#{name},#{money})")
    public void saveAccount(Account account);

}
```

 

### 2.7.3 **TestSpringMybatis**

【测试】可选测试

Service调用Dao

```java
@Test
public void testFindAllSpring() throws Exception {
    ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
    AccountDao accountDao = ac.getBean(AccountDao.class);
    List<Account> list = accountDao.findAll();
    for(Account account:list){
        System.out.println(account);
    }
}
```

 

### 2.7.4 **AccountServiceImpl.java**

在service中注入dao对象，添加 @Autowired

```java
/**
 * 账号的实现类
 */
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public List<Account> findAll() {
        System.out.println("业务层：查询所有的帐户...");
        return accountDao.findAll();
    }

    public void saveAccount(Account account) {
        accountDao.saveAccount(account);
    }

}
```

 

### 2.7.5 **AccountController.java**

添加 @Autowired，注入Service

```java
@Controller
@RequestMapping(path = "/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    /**
     * 查询所有
     * @return
     */
    @RequestMapping(path = "/findAll")
    public String findAll(Model model) {
        System.out.println("表现层：查询所有账户...");
        List<Account> list = accountService.findAll();
        System.out.println(list);
        model.addAttribute("list",list);
        return "success";
    }


    /**
     * 保存
     * @return
     */
    @RequestMapping(path = "/save")
    public String saveAccount(Account account) {
        accountService.saveAccount(account);
        return "success";
    }

}
```

 

### 2.7.6 **success.jsp**

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>访问成功</h1>
<c:forEach items="${list}" var="account">
    ${account.id}---
    ${account.name}---
    ${account.money}<br>
</c:forEach>
</body>
</html>
```



测试：

![img](./img/009.png) 

## 2.8 **配置Spring的声明式事务管理**

### 2.8.1 **applicationContext.xml**

方案一：Spring的声明式事务处理（配置文件xml）

【路径】

1：配置事务管理器 

2：配置事务通知（对切入点方法的细化）

3：配置AOP增强事务（配置切入点，通知关联切入点）

```xml
<!-- 1：配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--2：配置事务通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*" read-only="true"/>
        <tx:method name="save*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
        <tx:method name="update*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
        <tx:method name="delete*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
    </tx:attributes>
</tx:advice>

<!-- 3：配置AOP增强事务 -->
<aop:config>
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.itheima.ssm.service..*.*(..))"/>
</aop:config>
```

方案二==Spring的声明式事务处理（注解方式）==

第一步：applicationContext.xml

【路径】

1：配置事务管理器 

2：开启注解的方式使用事务

```xml
<!--使用注解的方式，配置spring的声明式事务处理-->
<!--1：配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!--2：开启注解的方式使用事务-->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

第二步：在Service的类上，配置@Transactional

```java
@Service
@Transactional
public class AccountServiceImpl implements AccountService {
}
```

 

### 2.8.2 **index.jsp**

测试保存帐户的方法

```html
<hr>
<form action="account/save" method="post">
    姓名：<input type="text" name="name"/><br/>
    金额：<input type="text" name="money"/><br/>
    <input type="submit" value="保存"/><br/>
</form>
```

 

### 2.8.3 **AccountController.java**

使用response重定向处理，返回值为void。

```java
/**
 * 保存
 * @return
 */
@RequestMapping(path = "/save")
public void saveAccount(Account account, HttpServletRequest request, HttpServletResponse response) throws Exception {
    System.out.println("表现层：保存账户...");
    accountService.saveAccount(account);
    response.sendRedirect(request.getContextPath()+"/account/findAll");
    return;
}
```

或者：

==建议使用redirect关键字处理重定向，返回值为String==。

```java
@RequestMapping(path = "/save")
public String saveAccount(Account account) throws Exception {
    System.out.println("表现层：保存账户...");
    accountService.saveAccount(account);
    return "redirect:findAll";
}
```

 

测试：

![img](./img/010.png) 

重定向到：

![img](./img/011.png) 

## 2.9 **让SpringMVC扫描，不让Spring扫描**

第一步：配置springmvc.xml

【路径】

加载springMVC的同时，同时加载spring，使用import标签的resource属性完成加载

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.itheima.ssm">

    </context:component-scan>

    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- JSP文件所在的目录 -->
        <property name="prefix" value="/WEB-INF/pages/" />
        <!-- 文件的后缀名 -->
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- 设置静态资源不过滤 -->
    <mvc:resources location="/css/" mapping="/css/**" />
    <mvc:resources location="/images/" mapping="/images/**" />
    <mvc:resources location="/js/" mapping="/js/**" />

    <!-- 开启对SpringMVC注解的支持 -->
    <mvc:annotation-driven />
    <!-- 加载springMVC的同时，同时加载spring -->
    <import resource="applicationContext.xml"></import>
</beans>
```

==需要使用<import resource="applicationContext.xml"></import>==

第二步：applicationContext.xml

【路径】

去掉所有的组件扫描配置context:component-scan 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop.xsd
   http://www.springframework.org/schema/tx
   http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 删除组件扫描的配置 -->

    <!-- 配置C3P0的连接池对象 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///itcastspringmvc" />
        <property name="user" value="root" />
        <property name="password" value="root" />
    </bean>

    <!-- 配置SqlSession的工厂 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 配置扫描dao的包 -->
    <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.ssm.dao"/>
    </bean>


    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置事务通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="save*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
            <tx:method name="update*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
            <tx:method name="delete*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
        </tx:attributes>
    </tx:advice>

    <!-- 配置AOP增强事务 -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.itheima.service..*.*(..))"/>
    </aop:config>
</beans>
```

 

第三步：删除web.xml中的监听器

```xml
<!-- 配置Spring的监听器，默认加载WEB-INF下的applicationContext.xml -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!-- 该监听器默认只加载WEB-INF下的applicationContext.xml配置文件，通过contextConfigLocation改变位置 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

###  【小结】

ssm整合完成

一步一步的完成，掌握整合的思路和方法，整合的目的

1：搭建spring的环境

2：搭建springmvc的环境

3：spring整合springmvc

4：搭建mybatis的环境

5：spring整合mybatis

6：搭建事务处理

7：测试SSM



配置文件：==applicationContext.xml==（数据源，事务处理，spring整合mybatis）、==springmvc.xml==（springmvc的配置）

配置文件：==web.xml==（springmvc的配置）

以上配置，配1次够用了。

项目开发：@Controller、@Service、@Repository就可以了，Dao中会写sql语句，页面

# 第3章 SSM用户注册登录案例

## 3.1 **ssm配置**

### 3.1.1 **web.xml**

```xml
<!-- 配置前端控制器：服务器启动必须加载，需要加载springmvc.xml配置文件 -->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 配置初始化参数，创建完DispatcherServlet对象，加载springmvc.xml配置文件 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!-- 服务器启动的时候，让DispatcherServlet对象创建 -->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

<!-- 配置解决中文乱码的过滤器 -->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

 

### 3.1.2 **applicationContext.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop.xsd
   http://www.springframework.org/schema/tx
   http://www.springframework.org/schema/tx/spring-tx.xsd">


    <!-- 1：Spring整合Mybatis -->
    <!-- 配置C3P0的连接池对象 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///itcastspringmvc" />
        <property name="user" value="root" />
        <property name="password" value="root" />
    </bean>

    <!-- 配置SqlSession的工厂 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 配置扫描dao的包 -->
    <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.ssm.dao"/>
    </bean>

    <!-- 2：spring的声明式事务处理 -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置事务通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="save*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
            <tx:method name="update*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
            <tx:method name="delete*" isolation="DEFAULT" propagation="REQUIRED" read-only="false"/>
        </tx:attributes>
    </tx:advice>

    <!-- 配置AOP增强事务 -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.itheima.ssm.service..*.*(..))"/>
    </aop:config>
</beans>
```

 

### 3.1.3 **springmvc.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.itheima.ssm"></context:component-scan>

    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- JSP文件所在的目录 -->
        <property name="prefix" value="/WEB-INF/pages/" />
        <!-- 文件的后缀名 -->
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- 设置静态资源不过滤 -->
    <!--<mvc:resources location="/css/" mapping="/css/**" />-->
    <!--<mvc:resources location="/img/" mapping="/img/**" />-->
    <!--<mvc:resources location="/plugins/" mapping="/plugins/**" />-->

    <!--静态资源一律不拦截，包括html-->
    <mvc:default-servlet-handler />


    <!-- 开启对SpringMVC注解的支持 -->
    <mvc:annotation-driven />

    <import resource="applicationContext.xml"></import>
</beans>
```

 

## 3.2 **用户注册**

### 【目标】

使用SSM完成用户注册功能（保存）

### 【路径】

1：数据库设计

2：创建工程，导入jar包

3：配置文件：applicationContext.xml，springmvc.xml，web.xml

4：创建实体User.java

5：导入页面（注册页面、登录页面）

6：创建UserController.java

7：创建接口UserService.java和实现类UserServiceImpl.java

8：创建接口UserDao.java

9：register.html测试

### 【讲解】

### 3.2.1 **数据库表设计**

> ​		表结构：
>
> ​			==number	varchar  // 登录名==
>
> ​			name	varchar  // 姓名
>
> ​			==password	varchar  // 密码==
>
> ​			mobile	varchar  // 手机号
>
> ​			qq	    varchar  // QQ
>
> ​			email	varchar  // 电子邮箱
>

```sql
create table `user` (
    `number` varchar(40) not null, # 主键
    `name` VARCHAR(20) DEFAULT NULL,
    `password` VARCHAR(60) DEFAULT NULL,
    `mobile` VARCHAR(11) DEFAULT NULL,
    `qq` VARCHAR(11) DEFAULT NULL,
    `email` VARCHAR(60) DEFAULT NULL,
    primary key (`number`)
) ENGINE=INNODB DEFAULT CHARSET = utf8;
```

登录名number作为主键，具有业务含义的主键叫做自然主键；我们把自增长的id主键称之为代理主键（有mysql代理主键）

### 3.2.2 **创建工程导入jar包**

​	

![img](./img/012.png) 

pom.xml

```xml
<packaging>war</packaging>
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <mysql.version>5.1.6</mysql.version>
    <mybatis.version>3.4.5</mybatis.version>
</properties>

<dependencies>

    <!-- spring -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.6.8</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>compile</scope>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <!-- log start -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <!-- log end -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.0</version>
    </dependency>

    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
        <type>jar</type>
        <scope>compile</scope>
    </dependency>

</dependencies>

<build>
    <finalName>ssm</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <showWarnings>true</showWarnings>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <!--插件使用的相关配置-->
            <configuration>
                <!--端口号-->
                <port>18881</port>
                <!--写当前项目的名字(虚拟路径),如果写/，那么每次访问项目就不需要加项目名字了-->
                <path>/</path>
                <!--解决get请求乱码-->
                <uriEncoding>UTF-8</uriEncoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

 tomcat7-maven-plugin：开发测试的时候，经常使用的插件，不需要使用外部tomcat进行发布了，使用tomcat7的插件完成。需要执行的命令tomcat7:run

### 3.2.3 **创建User.java**

```java
public class User implements Serializable {
    private String number;
    private String name;
    private String password;
    private String mobile;
    private String qq;
    private String email;
    // 提供set和get方法
}
```

 

### 3.2.4 **注册页面**

拷贝页面到项目。

![img](./img/013.png) 

### 3.2.5 **创建UserController.java**

页面数据发送到Controller，创建Controller

```java
@Controller
@RequestMapping(value = "/user")
public class UserController {

    @Autowired
    private UserService userService;

    /***
     * 增加用户
     * @param user
     * @return
     */
    @RequestMapping(value = "/save")
    public String save(User user){
        //增加
        int acount = userService.saveUser(user);

        if(acount>0){
            //新增成功，返回登录页面
            return "redirect:/login.html";
        }else{
            //新增失败，返回注册页面
            return "redirect:/register.html";
        }
    }

}
```

 

### 3.2.6 **创建接口UserService.java和实现类UserServiceImpl.java**

1：UserService.java

```java
public interface UserService {

    /***
     * 增加用户
     * @param user
     * @return
     */
    int saveUser(User user);
}
```

 2：UserServiceImpl.java

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    public int saveUser(User user) {
        return userDao.save(user);
    }
}
```

如果是注解的方式配置声明式事务处理，需要添加在UserServiceImpl的类上添加：@Transactional

### 3.2.7 **创建接口UserDao.java**

并编写SQL语句

```java
public interface UserDao {

    /***
     * 增加操作
     * @param user
     * @return
     */
    @Insert("insert into user(number,password,mobile)values(#{number},#{password},#{mobile})")
    int save(User user);

}
```

 

 

### 3.2.8 **register.html测试**

![img](./img/014.png) 

#### 3.2.8.1 **修改页面register.html**

```html
<form id="subform" action="/user/save" method="post" class="sui-form form-horizontal">
   <div class="control-group">
      <label class="control-label">用户名：</label>
      <div class="controls">
         <input type="text" name="number" placeholder="请输入你的用户名" class="input-xfat input-xlarge">
      </div>
   </div>
   <div class="control-group">
      <label for="inputPassword" class="control-label">登录密码：</label>
      <div class="controls">
         <input type="password" id="pwd" name="password" placeholder="设置登录密码" class="input-xfat input-xlarge">
      </div>
   </div>
   <div class="control-group">
      <label for="inputPassword" class="control-label">确认密码：</label>
      <div class="controls">
         <input type="password" id="rpwd" placeholder="再次确认密码" class="input-xfat input-xlarge">
      </div>
   </div>

   <div class="control-group">
      <label class="control-label">手机号：</label>
      <div class="controls">
         <input type="text" name="mobile" placeholder="请输入你的手机号" class="input-xfat input-xlarge">
      </div>
   </div>

   <div class="control-group">
      <label for="inputPassword" class="control-label">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</label>
      <div class="controls">
         <input name="m1" type="checkbox" value="2" checked=""><span>同意协议并注册《品优购用户协议》</span>
      </div>
   </div>
   <div class="control-group">
      <label class="control-label"></label>
      <div class="controls btn-reg">
         <button class="sui-btn btn-block btn-xlarge btn-danger" type="button" id="btn">完成注册</button>
      </div>
   </div>
</form>
```

 

#### 3.2.8.2 **实现密码输入框的页面校验**

register.html

```html
<script src="plugins/jQuery/jquery-2.2.3.min.js"></script>
<script>
    $(function () {
        /*
        * 点击注册按钮
        *     判断2次密码是否一致
        *     如果一致，提交到后台，否则提示错误
        * */
        $('#btn').click(function () {
            //获取2次密码
            var pwd = $('#pwd').val();

            var rpwd=$('#rpwd').val();

            if(pwd==rpwd){
                //提交到后台
                $('#subform').submit();
            }else{
                //提示
                alert("2次密码输入有误")
            }
        });

    });
</script>
```

 

#### 3.2.8.3 **启动tomcat**

测试使用外部的tomcat完成

![img](./img/015.png) 

![img](./img/016.png) 

测试：使用tomcat插件（tomcat7-maven-plugin）

command line：tomcat7:run

![img](./img/017.png)  

 

测试：保存成功

![img](./img/018.png) 

###  【小结】

搭建框架

 1：数据库

2：工程，创建工程，导入jar包

3：配置文件：applicationContext.xml，springmvc.xml，web.xml

4：创建实体User.java

5：创建UserController.java

6：创建接口UserService.java和实现类UserServiceImpl.java

7：创建接口UserDao.java

8：导入页面（注册页面、登录页面）。register.html测试

## 3.3 **用户登录**

### 【目标】

使用SSM完成用户登录功能

### 【路径】

1：导入登录页面login.html

2：编写UserController.java（查询，使用用户名和密码）

3：编写UserServiceImpl.java

4：编写UserDao.java

5：登录成功（重定向到欢迎页，WEB-INF/pages/index.jsp）

6：WelcomeController.java（跳转到欢迎页）

### 【讲解】

### 3.3.1 **login.html登录页面**

```html
<form class="sui-form" action="/user/login" method="post">
   <div class="input-prepend"><span class="add-on loginname"></span>
      <input id="prependedInput" type="text" name="number" placeholder="邮箱/用户名/手机号" class="span2 input-xfat">
   </div>
   <div class="input-prepend"><span class="add-on loginpwd"></span>
      <input id="prependedInput" type="password" name="password" placeholder="请输入密码" class="span2 input-xfat">
   </div>
   <div class="setting">
      <label class="checkbox inline"><input name="m1" type="checkbox" value="2" checked="">自动登录</label>
      <span class="forget">忘记密码？</span>
   </div>
   <div class="logined">
      <button class="sui-btn btn-block btn-xlarge btn-danger" type="submit">登&nbsp;&nbsp;录</button>
   </div>
</form>
```

 

### 3.3.2 **UserController.java**

```java
/***
 * 用户登录
 * @return
 */
@RequestMapping(value = "/login")
public String login(User user, HttpSession session){
    //数据库查询登录
    User loginUser = userService.findByNumberAndPassword(user);

    //判断loginUser是否为空
    if(loginUser!=null){
        //登录成功跳转到欢迎页
        session.setAttribute("loginUser",user);
        return "redirect:/index/welcome";
    }else{
        //为空，登录失败，继续登录
        return "redirect:/login.html";
    }
}
```

 

### 3.3.3 **UserServiceImpl.java**

```java
public User findByNumberAndPassword(User user) {
    return userDao.findByNumberAndPassword(user);
}
```

 

### 3.3.4 **UserDao.java**

```java
/****
 * 登录
 * @param user
 * @return
 */
@Select("select * from user where number=#{number} and password=#{password}")
User findByNumberAndPassword(User user);
```

 

或者：@Param注解

```java
@Select(value = "select * from user where number = #{number} and password=#{password}")
User findByNumberAndPassword(@Param("number") String number, @Param("password") String password);
```

或者：map集合

```java
/**
 *       Map<String,Object> map = new HashMap<>();
         map.put("number",user.getNumber());
         map.put("password",user.getPassword());
 */
@Select(value = "SELECT * FROM USER WHERE number = #{number} AND password = #{password}")
User findByNumberAndPassword(Map<String, Object> map);
```

 

### 3.3.5 **登录成功--->重定向到欢迎页**

#### 3.3.5.1 **WelcomeController.java**

```java
@Controller
@RequestMapping(value = "/index")
public class WelcomeController {

    /***
     * 欢迎页
     * @return
     */
    @RequestMapping(value = "/welcome")
    public String welcome(){
        return "index";
    }
}
```

 

#### 3.3.5.2 **index.html**

将admin下的index.html拷贝到WEB-INF\pages下

同时创建index.jsp，将index.html的内容放置到index.jsp

![img](./img/019.png) 

![img](./img/020.png)  

在index.jsp中获取登录信息

```html
<!-- Sidebar toggle button-->
<a href="#" class="sidebar-toggle" data-toggle="offcanvas" role="button">
   <span class="sr-only">Toggle navigation</span>
</a>
欢迎你：${sessionScope.loginUser.number}
```

 

![img](./img/021.png) 

###  【小结】

1：导入登录页面login.html

2：编写UserController.java

3：编写UserServiceImpl.java

4：编写UserDao.java（select * from user where number = #{number} and password=#{password}）

5：登录成功（重定向到欢迎页index.jsp）

6：WelcomeController.java（跳转到欢迎页）

## 3.4 整合拦截器

### 【目标】

【需求】：用户如果已经登录，正常访问

​		  用户如果没有登录，则不能访问欢迎页

### 【路径】

1：声明拦截器，LoginInterceptor.java

2：配置拦截器，springmvc.xml

### 【讲解】

### 3.4.1 **LoginInterceptor.java**

第一步：声明拦截器

创建包com.itheima.ssm.interceptor

创建类LoginInterceptor.java

```java
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        /****
         * 用户是否在线
         * 如果不在线，不允许访问欢迎页
         */
        Object loginUser = request.getSession().getAttribute("loginUser");

        if(loginUser==null){
            response.sendRedirect(request.getContextPath()+"/login.html");
            return false;
        }

        return true;
    }
}
```

 

### 3.4.2 **springmvc.xml**

第二步：配置拦截器

【路径】

1：指定拦截路径

2：指定处理请求的拦截器

```xml
<!--配置拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--1：指定拦截路径-->
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/*.html"></mvc:exclude-mapping>
        <mvc:exclude-mapping path="/**/*.js"></mvc:exclude-mapping>
        <mvc:exclude-mapping path="/img/**/*.*"></mvc:exclude-mapping>
        <mvc:exclude-mapping path="/css/**/*.*"></mvc:exclude-mapping>
        <mvc:exclude-mapping path="/user/login"></mvc:exclude-mapping>
        <mvc:exclude-mapping path="/user/save"></mvc:exclude-mapping>

        <!--
           2：指定处理请求的拦截器
        -->
        <bean class="com.itheima.ssm.interceptor.LoginInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```

【测试】重新启动，在没有Session的情况下，访问/index/welcome，此时重定向到登录页面。

###  【小结】

拦截器的作用：在访问Controller类之前，做出判断

拦截器的使用：

1：声明拦截器，LoginInterceptor.java

2：配置拦截器，springmvc.xml