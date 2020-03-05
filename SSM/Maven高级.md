maven高级

学习目标：

1：jar包冲突（解决方案：直接排除，直接依赖）

2：工程分解（一个父工程，四（n）个子工程）  --重点

3：私服（了解）



# Maven-教案-实战(IDEA)

# 1.  第一章：**[**课程大纲](#课程大纲)

### 【目标】

回顾Maven基础课程

### 【路径】

1：Maven好处

2：安装配置Maven

3：三种仓库

4：常见命令

5：坐标的书写规范

6：如何添加坐标

7：依赖范围

### 【讲解】

## 1.1.  回顾[理解]

什么是Maven？

![img](./img/001.png) 

### 1.1.1. Maven好处

1. 节省磁盘空间

2. 可以一键构建

![img](./img/002.png) 

3. 可以跨平台

4. 应用在大型项目时可以提高开发效率（jar包管理，maven的工程分解，可分模块开发）

### 1.1.2. 安装配置 maven

注意： 3.3+版本需要 jdkj.7+以上的支持

### 1.1.3. 三种仓库

1. 本地仓库

2. 远程仓库（私服）

3. 中央仓库

### 1.1.4. 常见的命令

1. Compile
2. Test
3. Package（jar，war）
4. Install（安装到本地仓库）
5. Deploy（部署到远程仓库（私服））

6. Clean（target下生成的文件）

### 1.1.5. 坐标的书写规范

1. groupId 公司或组织域名的倒序

2. artifactId 项目名或模块名

3. version 版本号

![img](./img/003.png) 

### 1.1.6. 如何添加坐标

1.在本地仓库中搜索

2.互联网上搜，推荐网址 <http://www.mvnrepository.com/>

![img](./img/004.png) 

### 1.1.7. 依赖范围

<scope>属性：

1. Compile（默认）


2. Test


3. Provided


4. Runtime

![img](./img/005.png) 

例如：

![img](./img/006.png) 

### 【小结】

1：Maven好处（了解）

2：安装配置Maven（熟练）

3：三种仓库（本地仓库，中央仓库（远程仓库），私服（代理仓库））

4：常见命令（clean compile test package install deploy）

5：坐标的书写规范（groupId artifactId version）

6：如何添加坐标（http://www.mvnrepository.com/）

7：依赖范围

# 2.  第二章：**[**Maven]中的jar包冲突问题

### 【目标】

使用maven依赖导入jar包，会不会出现导入相同的jar包，且版本不一致呢？

### 【路径】

1：演示jar包冲突问题：遵循第一声明者优先原则

2：解决jar包冲突问题，路径近者优先

3：解决jar包冲突问题，直接排除法

### 【讲解】

创建一个jar包工程，流程如下图：

![img](./img/007.png) 

## 2.1.  jar包冲突（问题）：第一声明优先原则

​	哪个jar包在靠上的位置，这个jar包就是先声明的，先声明的jar包下的依赖包，可以优先引入项目中。

​	我们在pom.xml中引入如下坐标，分别是spring中不同的版本。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>maven_day01_demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>jar</packaging>

    <!--导入相关依赖包-->
    <dependencies>
        <!--引入spring-context，它所以来的包都会导入进来-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>4.2.4.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

 

我们在控制面板的maven面板，点击查看依赖关系按钮，看到了包和包之间的依赖关系存在冲突，都使用了spring-core包，关系图如下：

![img](./img/008.png) 

我们再来看看他们依赖包的导入，发现导入的包却没有问题，包使用的都是5.0.2的版本。

![img](./img/009.png) 

我们把上面2个包的顺序调换后就变成了低版本的依赖导入。

![img](./img/010.png) 

问题：整合项目需要用到5的版本，引入4的版本，会不会出现异常（类没有找到，方法没有找到）

解决方案：？

## 2.2. jar包冲突（方案一）：路径近者优先原则（直接依赖）

直接依赖比传递依赖路径近，你那么最终进入项目的jar包会是路径近的直接依赖包。

直接依赖：项目中直接导入的jar包就是项目的直接依赖包。

传递依赖（间接依赖）：项目中没有直接导入的jar包，可以通过中直接依赖包传递到项目中去。

修改jar包，==直接引入依赖spring-core==

```xml
<!--导入相关依赖包-->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.2.4.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.2.RELEASE</version>
    </dependency>

    <!--引入直接依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.2.8.RELEASE</version>
    </dependency>
</dependencies>
```

 

此时优先引入的是直接依赖的引用

![img](./img/011.png)![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml2660\wps44.png) 

## 2.3. jar包冲突（方案二）：直接排除法

当我们需要排除某个jar包的依赖时，在配置exclusions标签的时候，内部可以不写版本号。pom.xml依赖如下：

```xml
<!--导入相关依赖包-->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.2.4.RELEASE</version>
        <!--直接排除-->
        <exclusions>
            <exclusion>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

快捷操作：

![img](./img/012.png) 

依赖导入的jar包如下：

没有添加exclusion之前

![img](./img/013.png) 

添加exclusion之后，因为排除了4.2.4的版本spring-core的jar包

![img](./img/014.png) 

### 【小结】

真实项目中，出现1个项目存在多个同种jar包的时候，需要我们进行解决maven的jar包冲突问题（异常：Class not found、Method not found等）

1：直接依赖原则（路径近者优先）

2：直接排除原则

# 3. [第三章：工程分层]

回顾之前项目开发：

![img](./img/015.png) 

工程分层后的开发：所有的service和dao的代码都在一起，==1：增强程序的通用性，2：降低了代码的耦合性==

![img](./img/016.png) 

### 【目标】

实现项目工程分解，掌握聚合和继承的作用

### 【路径】

1：聚合和继承

2：准备数据库环境

3：ssm_parent（父工程）

4：ssm_model（子工程）

5：ssm_dao（子工程）

6：ssm_service（子工程）

7：ssm_web（子工程）

8：测试（工程发布tomcat）

### 【讲解】

## 3.1. 聚合（多模块）和继承

 

继承和聚合结构图：

## ![img](./img/017.png) 

==特点1（继承）：==

==parent父工程：存放项目的所有jar包。==

==ssm_web和ssm_service和ssm_dao有选择的继承jar包，并在自己的工程中使用。这样可以消除jar包重复，并锁定版本==

==特点2（聚合）：==

​        ==ssm_web依赖于ssm_service，ssm_service依赖于ssm_dao，我们启动ssm_web，便可以访问我们的程序。==

​	==执行安装的时候，执行ssm_parent，就可以将所有的子工程全部进行安装。==

理解继承和聚合总结：

​	通常继承和聚合同时使用。

· 何为继承？

​	继承是为了消除重复，如果将 dao、 service、 web 分开创建独立的工程则每个工程的 pom.xml 文件中的内容存在重复，比如：设置编译版本、锁定 spring 的版本的等，可以将这些重复的 配置提取出来在父工程的 pom.xml 中定义。

· 何为聚合？

​	项目开发通常是分组分模块开发， 每个模块开发完成要运行整个工程需要将每个模块聚合在 一起运行，比如： dao、 service、 web 三个工程最终会打一个独立的 war 运行。

![img](./img/018.png) 

## 3.2. 准备数据库环境

准备工作，导入sql

创建数据库itcastmaven

执行items.sql

![img](./img/019.png) 

```sql
SET FOREIGN_KEY_CHECKS=0;
-- ----------------------------
-- Table structure for `items`
-- ----------------------------
DROP TABLE IF EXISTS `items`;
CREATE TABLE `items` (
  `id` int(10) NOT NULL auto_increment,
  `name` varchar(20) default NULL,
  `price` float(10,0) default NULL,
  `pic` varchar(40) default NULL,
  `createtime` datetime default NULL,
  `detail` varchar(200) default NULL,
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of items
-- ----------------------------
INSERT INTO `items` VALUES ('1', '传智播客web课程', '1000', null, '2018-03-13 09:29:30', '带我走上人生巅峰');
INSERT INTO `items` VALUES ('2', '黑马之路', null, null, '2018-03-28 10:05:52', '插入测试');
INSERT INTO `items` VALUES ('3', '黑马之路二', '199', null, '2018-03-07 10:08:04', '插入测试');
INSERT INTO `items` VALUES ('7', '插入测试', null, null, null, null);
INSERT INTO `items` VALUES ('8', '插入测试', null, null, null, null);
```

![img](./img/020.png) 

## 3.3. ssm_parent

![img](./img/021.png) 

删除src文件夹

![img](./img/022.png) 

### 3.3.1. pom.xml

 父工程：<packaging>pom</packaging>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm_parent</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <!--
        特殊属性定义，一般是版本号
    -->
    <properties>
        <spring.version>5.0.2.RELEASE</spring.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <mysql.version>5.1.6</mysql.version>
        <mybatis.version>3.4.5</mybatis.version>
        <aspectjweaver.version>1.6.8</aspectjweaver.version>
        <junit.version>4.12</junit.version>
        <jsp-api.version>2.0</jsp-api.version>
        <servlet-api.version>2.5</servlet-api.version>
        <jstl.version>1.2</jstl.version>
        <mybatis-spring.version>1.3.0</mybatis-spring.version>
        <druid.version>1.0.9</druid.version>
        <!--文件的编码格式-->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

    <!--
        jar包管理
        dependencyManagement:并非导入依赖，而只是管理依赖（这样子工程可供选择）
    -->
    <dependencyManagement>
        <!--引入依赖-->
        <dependencies>
            <!-- spring（切面） -->
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjweaver</artifactId>
                <version>${aspectjweaver.version}</version>
            </dependency>
			<!-- spring（aop） -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aop</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <!--spring包（核心）-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <!--用于SpringMVC-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <!--用于数据库源相关操作-->
            <!-- spring（整合jdbc） -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId>
                <version>${spring.version}</version>
            </dependency>
			<!-- spring（事务） -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-tx</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <!--Servlet相关API（可以使用Request、Response）-->
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>servlet-api</artifactId>
                <version>${servlet-api.version}</version>
                <scope>provided</scope>
            </dependency>

            <dependency>
                <groupId>javax.servlet.jsp</groupId>
                <artifactId>jsp-api</artifactId>
                <version>${jsp-api.version}</version>
                <scope>provided</scope>
            </dependency>

            <!--jstl标签-->
            <dependency>
                <groupId>jstl</groupId>
                <artifactId>jstl</artifactId>
                <version>${jstl.version}</version>
            </dependency>

            <!--MySQL数据库驱动-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
				<scope>runtime</scope>
            </dependency>

            <!--spring测试-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>


            <!-- log日志 start -->
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

            <!--mybatis-->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>

            <!--MyBatis集成Spring-->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>${mybatis-spring.version}</version>
            </dependency>

            <!--数据源-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>


    <build>
        <pluginManagement>
            <!--插件-->
            <plugins>
                <!--tomcat插件-->
                <plugin>
                    <groupId>org.apache.tomcat.maven</groupId>
                    <artifactId>tomcat7-maven-plugin</artifactId>
                    <version>2.2</version>
                    <!--插件使用的相关配置-->
                    <configuration>
                        <!--端口号-->
                        <port>18081</port>
              <!--写当前项目的名字(虚拟路径),如果写/，那么每次访问项目就不需要加项目名字了-->
                        <path>/</path>
                        <!--解决get请求乱码-->
                        <uriEncoding>UTF-8</uriEncoding>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>
```

 【小结】

- <package>pom</pcakage>

* 管理所有的jar包（并锁定版本）
* 父工程不需要开发的代码，src的文件夹可以删除

## 3.4. ssm_model

【路径】

1：配置pom.xml

2：创建Items.java

选择Module

![img](./img/023.png) 

![img](./img/024.png) 

对应数据库的表和字段

![img](./img/025.png) 

### 3.4.1. pom.xml

查看子工程pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>ssm_parent</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ssm_model</artifactId>
    <packaging>jar</packaging>

</project>
```

查看父工程pom.xml

表示==聚合了ssm_model==

```xml
<modules>
    <module>ssm_model</module>
</modules>
```

 

### 3.4.2. 创建Items.java

```java
public class Items {

    private Integer id;
    private String name;
    private Float price;
    private String pic;
    private Date createtime;
    private String detail;
    //get..set..
}
```

【小结】

* <package>jar</pcakage>
* 管理项目中所有的实体类

## 3.5. ssm_dao

【路径】

1：配置pom.xml

2：创建spring-mybatis.xml（spring整合mybatis的配置）

3：创建ItemsDao.java

4：创建ItemsDao.xml

5：创建mybatis.xml（mybatis的配置文件）（可以省略）

![img](./img/026.png) 

### 3.5.1. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>ssm_parent</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ssm_dao</artifactId>

    <!--jar包-->
    <packaging>jar</packaging>

    <!--引入依赖-->
    <dependencies>
        <!--model的依赖-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>ssm_model</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
        </dependency>

        <!--MyBatis集成Spring-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
        </dependency>

        <!--数据源-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>

        <!--MySQL数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!--SpringJdbc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
        </dependency>

        <!-- log start -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
        <!-- log end -->
    </dependencies>
</project>
```

查看父工程pom.xml

```xml
<modules>
    <module>ssm_model</module>
    <module>ssm_dao</module>
</modules>
```

 

### 3.5.2. 创建spring-mybatis.xml

【路径】

1：数据库连接池

2：SqlSessionFactoryBean

3：Dao层接口扫描，让Dao被spring管理

在resources下创建spring-mybatis.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <!-- 1：数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/itcastmaven?characterEncoding=utf8" />
        <property name="username" value="root" />
        <property name="password" value="root" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>

    <!--
        2：SqlSessionFactoryBean
    -->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--
            指定数据源
        -->
        <property name="dataSource" ref="dataSource" />

        <!--
            指定别名配置
        -->
        <property name="typeAliasesPackage" value="com.itheima.ssm.domain" />

        <!--
            MyBatis核心配置文件
            延迟加载、缓存...
            SqlMapConfig.xml
        
        <property name="configLocation" value="classpath:mybatis.xml" />-->

        <!--
        指定映射文件的位置
        <property name="mapperLocations">
            <array>
                <value>classpath:com/itheima/ssm/dao/*Dao.xml</value>
            </array>
        </property>-->
    </bean>


    <!--
       3：Dao层接口扫描，让Dao被spring管理
    -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--指定Dao接口的包-->
        <property name="basePackage" value="com.itheima.ssm.dao" />
    </bean>

</beans>
```



### 3.5.3. 创建ItemsDao.java

```java
public interface ItemsDao {

    /***
     * 查询所有
     * @return
     */
    List<Items> list();

    /***
     * 保存操作
     * @param items
     * @return
     */
    int save(Items items);
}
```

 

### 3.5.4. 创建ItemsDao.xml

在com/itheima/ssm/dao/ItemsDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    namespace="Dao接口的全限定名"
-->
<mapper namespace="com.itheima.ssm.dao.ItemsDao">


    <!--保存操作-->
    <insert id="save" parameterType="Items">
        INSERT  INTO items(name,price,pic,createtime,detail) VALUES(#{name},#{price},#{pic},#{createtime},#{detail})
    </insert>

    <!--查询所有-->
    <select id="list" resultType="Items">
        SELECT * FROM  items
    </select>


</mapper>
```

 

### 3.5.5. 创建mybatis.xml（可以省略）

在resources下创建mybatis.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
```

 【小结】

* <package>jar</pcakage>
* 引入依赖     ssm_model     、mybatis的依赖...
* spring-mybatis.xml（spring的配置文件）（spring整合mybatis的框架）
* 接口ItemsDao.java
* 创建映射文件ItemsDao.xml

## 3.6. ssm_service

【路径】

1：pom.xml

2：创建spring-service.xml（spring的声明式事务处理）

3：创建ItemsService接口

4：创建ItemsServiceImpl实现类

![img](./img/027.png) 

### 3.6.1. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>ssm_parent</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ssm_service</artifactId>

    <!--jar包-->
    <packaging>jar</packaging>

    <!--依赖-->
    <dependencies>
        <!--依赖dao-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>ssm_dao</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!-- spring -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
        </dependency>
    </dependencies>
</project>
```

查看父工程pom.xml

```xml
<modules>
    <module>ssm_model</module>
    <module>ssm_dao</module>
    <module>ssm_service</module>
</modules>
```

 

### 3.6.2. 创建spring-service.xml

【路径】

1：创建一个事务管理器

2：配置事务的通知，及传播特性，对切入点方法的细化

3：AOP声明式事务配置（配置切入点，通知关联切入点）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">


    <!--1：创建一个事务管理器-->
    <bean id="txtManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--指定数据源-->
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--2：配置事务的传播特性，对切入点方法的细化-->
    <tx:advice id="txAdvice" transaction-manager="txtManager">
        <!--配置传播特性属性-->
        <tx:attributes>
            <!--
                对应方法参与事务并且在事务下执行，事务隔离剂别使用默认隔离级别,发生异常需要事务回滚
            -->
            <tx:method name="add*" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.Exception" />
            <tx:method name="insert*" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.Exception" />
            <tx:method name="save*" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.Exception" />
            <tx:method name="delete*" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.Exception" />
            <tx:method name="update*" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.Exception" />
            <tx:method name="edit*" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.Exception" />

            <!--
                只读操作
            -->
            <tx:method name="*" read-only="true" />
        </tx:attributes>
    </tx:advice>

    <!--
        3：AOP声明式事务配置（配置切入点，通知关联切入点）
    -->
    <aop:config>
        <!--切点指点-->
        <aop:pointcut id="tranpointcut" expression="execution(* com.itheima.ssm.service..*.*(..))" />

        <!--配置通知-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="tranpointcut" />
    </aop:config>



    <!--引入spring-mybatis.xml-->
    <import resource="spring-mybatis.xml" />
</beans>
```

 

### 3.6.3. 创建ItemsService接口

```java
public interface ItemsService {

    /***
     * 列表查询
     * @return
     */
    List<Items> list();

    /***
     * 增加商品
     * @param items
     * @return
     */
    int save(Items items);
}
```

 

### 3.6.4. 创建ItemsServiceImpl

```java
@Service
public class ItemsServiceImpl implements ItemsService {

    @Autowired
    private ItemsDao itemsDao;

    /***
     * 集合查询
     * @return
     */
    public List<Items> list() {
        return itemsDao.list();
    }

    /***
     * 增加商品测试事务
     * @param items
     * @return
     */
    public int save(Items items) {
        int acount = itemsDao.save(items);

        System.out.println("acount:"+acount);

        //测试事务，如果出错，是否回滚
        //int q=10/0;

        return acount;
    }
}
```

 【小结】

* <package>jar</pcakage>
* 引入依赖   ssm_dao   、spring相关的依赖
* spring-service.xml（声明式事务处理）
* 接口ItemsService.java
* 实现类ItemsServiceImpl.java

## 3.7. ssm_web

【路径】

1：pom.xml

2：创建web.xml

3：创建springmvc.xml

4：创建ItemsController.java

5：创建页面items.jsp

有的公司会命名为ssm_controller，都可以。

![img](./img/028.png) 

### 3.7.1. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>ssm_parent</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ssm_web</artifactId>

    <!--war包-->
    <packaging>war</packaging>

    <!--依赖引入-->
    <dependencies>
        <!--依赖service-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>ssm_service</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

	    <!--导入springmvc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>

        <!--servletAPI -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <scope>provided</scope>
        </dependency>
        
        <!--jstl表达式 -->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
        </dependency>

    </dependencies>


    <build>
        <!--插件-->
        <plugins>
            <!--tomcat插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <!--插件使用的相关配置-->
                <configuration>
                    <!--端口号-->
                    <port>18081</port>
                    <!--写当前项目的名字(虚拟路径),如果写/，那么每次访问项目就不需要加项目名字了-->
                    <path>/</path>
                    <!--解决get请求乱码-->
                    <uriEncoding>UTF-8</uriEncoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

查看父工程pom.xml

```xml
<modules>
    <module>ssm_model</module>
    <module>ssm_dao</module>
    <module>ssm_service</module>
    <module>ssm_web</module>
</modules>
```

 

### 3.7.2.创建web.xml

【路径】

1:配置编码过滤器

2:springmvc前端核心控制器

![img](./img/029.png) 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--
        1:配置编码过滤器
    -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

        <!--指定编码-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>

    <!--拦截映射-->
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>


    <!--
        2:springmvc前端核心控制器
        创建了SpringMVC容器
    -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--指定springmvc配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        
        <!--启动加载优先级-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <!--指定映射拦截-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

 

### 3.7.3. 创建springmvc.xml

【路径】

1：包扫描

2：视图解析器

3：springmvc注解驱动，自动配置mvc的处理器适配器和处理映射器

4：静态资源不过滤

5：import导入spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--1：包扫描-->
    <context:component-scan base-package="com.itheima" />

    <!--2：视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/" />
        <property name="suffix" value=".jsp" />
    </bean>

    <!--3：springmvc注解驱动，自动配置mvc的处理器适配器和处理映射器-->
    <mvc:annotation-driven />

    <!--4：静态资源不过滤-->
    <mvc:default-servlet-handler />


    <!--5：导入spring.xml-->
    <import resource="spring-service.xml" />

</beans>
```

 

### 3.7.4. 创建ItemsController

```java
@Controller
@RequestMapping(value = "/items")
public class ItemsController {


    @Autowired
    private ItemsService itemsService;


    /*****
     * 列表访问
     * @return
     */
    @RequestMapping(value = "/list")
    public String list(Model model){
        //集合查询
        List<Items> items = itemsService.list();
        //存入回显
        model.addAttribute("items",items);
        return "items";
    }

    /***
     * 事务测试
     * 增加商品
     * @return
     */
    @RequestMapping(value = "/save")
    public String save(Items items){
       int acount =  itemsService.save(items);
       //集合列表页跳转
       return "redirect:/items/list";
    }

}
```

 

### 3.7.5. 创建页面items.jsp

在/WEB-INF/pages/创建items.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>新增/查询</title>
</head>
<body>
<table>
    <form action="/items/save" method="post">
        <table>
            <tr>
                <td>名称</td>
                <td><input type="text" name="name"/></td>
            </tr>
            <tr>
                <td>价格</td>
                <td><input type="text" name="price"/></td>
            </tr>
            <tr>
                <td>图片</td>
                <td><input type="text" name="pic"/></td>
            </tr>
            <tr>
                <td>创建日期</td>
                <td><input type="text" name="createtime"/></td>
            </tr>
            <tr>
                <td>详情</td>
                <td><input type="text" name="detail"/></td>
            </tr>
            <tr>
                <td colspan="2">
                    <input type="submit" value="提交"/>
                </td>
            </tr>
        </table>
    </form>
</table>
<hr>
<table border="1">
    <tr>
        <td>ID</td>
        <td>name</td>
        <td>price</td>
        <td>pic</td>
        <td>createTime</td>
        <td>detail</td>
        <td></td>
    </tr>
    <c:forEach items="${items}" var="item">
        <tr>
            <td>${item.id}</td>
            <td>${item.name}</td>
            <td>${item.price}</td>
            <td>${item.pic}</td>
            <td>${item.createtime}</td>
            <td>${item.detail}</td>
        </tr>
    </c:forEach>
</table>

</body>
</html>
```

 

 【小结】

* <package>war</pcakage>
* 引入依赖    ssm_service   、 springmvc   、 servlet_api等
* springmvc.xml（springmvc的相关配置）
* web.xml（springmvc的核心控制器、post乱码过滤器）
* 类ItemsController.java
* 测试页面items.jsp

## 3.8. 测试

【路径】

1：测试打包package、安装install（演示maven聚合）

2：发布到外部tomcat

3：使用maven的tomcat插件发布

### 3.8.1. 打包package、安装install（聚合的作用）

在父工程打包

![img](./img/030.png) 

所有的子工程都会被打包（package）。这就是“聚合”的作用。

也可以同时安装（install），同时部署（deploy）

![img](./img/031.png) 

### 3.8.2. 发布到外部tomcat

![img](./img/032.png) 

![img](./img/033.png) 

使用http://localhost:18081/ssm/items/list测试：

![img](./img/034.png) 

【小结】

当使用外部tomcat进行开发的时候，不需要将ssm_parent，ssm_model，ssm_dao，ssm_service安装到本地仓库。

### 3.8.3.  使用maven的tomcat插件发布

![img](./img/035.png) 

 

使用http://localhost:18081/items/list测试：

![img](./img/036.png) 

【小结】

使用maven内置的tomcat插件的时候 ：

第一种：配置D:\ideaProjects\ssm_parent：parent是有聚合的功能，不需要将ssm_parent，ssm_model，ssm_dao，ssm_service安装到本地仓库。

第二种：配置D:\ideaProjects\ssm_parent\ssm_web：需要将ssm_parent，ssm_model，ssm_dao，ssm_service安装到本地仓库。

### 【小结】

1：聚合和继承

2：准备数据库环境

3：ssm_parent（父工程）

4：ssm_model（子工程）

5：ssm_dao（子工程）

6：ssm_service（子工程）

7：ssm_web（子工程）

8：测试（工程发布tomcat）

# 4. 第四章：maven 私服[了解]

### 【目标】

公司内部搭建maven私服，从私服上下载jar包，本地仓库不连接中央仓库，而是使用私服去连接中央仓库

### 【路径】

1：私服作用

2：需求（将jar包上传到私服，下载私服的jar包到本地）

3：私服分析

4：搭建私服环境

5：私服安装后配置

6：将项目发布到私服

7：从私服下载jar包到本地

8：把第三方jar包放入本地仓库或私服

9：阿里私服（介绍）

### 【讲解】

## 4.1. 私服作用

私服作用一：下载速度快

![img](./img/037.png) 

私服作用二：通用

![img](./img/038.png) 

## 4.2. 需求

正式开发，不同的项目组开发不同的工程。
【需求】：==ssm-dao== 工程开发完毕，==发布到私服==。

【需求】：ssm-service 从私服==下载 ssm-dao==使用。

## 4.3. 分析

​	==公司在自己的局域网内搭建自己的远程仓库服务器，称为私服==， 私服服务器即是公司内部的 maven 远程仓库， 每个员工的电脑上安装 maven 软件并且连接私服服务器，员工将自己开发的项目打成 jar 并发布到私服服务器，其它项目组从私服服务器下载所依赖的构件（jar）。私服还充当一个代理服务器，当私服上没有 jar 包会从互联网中央仓库自动下载，如下图：

![img](./img/039.png) 

## 4.4. 搭建私服环境（了解）

【路径】

1：下载私服nexus

2：安装nexus

3：nexus仓库类型介绍

### 4.4.1. 下载 nexus

Nexus 是 Maven 仓库管理器， 通过 nexus 可以搭建 maven 仓库，同时 nexus 还提供强大的仓库管理功能，构件搜索功能等。
下载 Nexus， 下载地址：<https://www.sonatype.com/download-oss-sonatype>

![img](./img/040.png) 

我们今天将使用nexus-2.12.0-01-bundle.zip的nexus版本。

![img](./img/041.png) 

### 4.4.2. 安装 nexus

使用zip

1：安装nexus

第一步：将nexus-2.12.0-01-bundle.zip解压，将nexus-2.12.0-01拷贝到D盘的software文件夹下（不要出现中文和空格）

![img](./img/042.png) 

拷贝到D盘/software下，并解压

![img](./img/043.png) 

nexus-2.12.0-01：私服安装目录（配置）

sonatype-work：私服工作目录（索引和仓库）

 

第二步：安装nexus服务

​	进入nexus-2.12.0-01\bin目录，然后使用nexus.bat命令进行操作（记得以管理员身份运行）

​	命令：nexus.bat install

 

![img](./img/044.png) 

 

 

第三步：启动nexus服务

方式1、直接启动nexus服务

​	![img](./img/045.png)

 

方式2：使用命令启动

命令：nexus.bat start

启动成功。

![img](./img/046.png) 

 

注意：启动的时候报错

![img](./img/01.png)

解决方案：

1：查找D:\software\nexus-2.12.0-01-bundle\nexus-2.12.0-01\bin\jsw\conf中的wrapper.conf

2：修改wrapper.conf的配置：

```
wrapper.java.command=D:\Program Files\Java\jdk1.8.0_172\bin\java
```

第四步：访问nexus

地址：<http://localhost:8081/nexus> 

点击右上角的login，可以登录

用户名：admin

密码：admin123

![img](./img/047.png) 

 

 

2：卸载nexus

cmd进入nexus的bin目录，

停止服务

执行：nexus.bat stop

执行：nexus.bat uninstall

![img](./img/048.png) 

查看window服务列表nexus已被删除。

 

 

3：查看nexus

查看nexus的配置文件conf/nexus.properties

![img](./img/049.png) 

![img](./img/050.png) 

```properties
\# Jetty section

application-port=8081  	# nexus的访问端口配置

application-host=0.0.0.0 	# nexus主机监听配置(不用修改)

nexus-webapp=${bundleBasedir}/nexus 	# nexus工程目录

nexus-webapp-context-path=/nexus	 # nexus的web访问路径

 
\# Nexus section

nexus-work=${bundleBasedir}/../sonatype-work/nexus   # nexus仓库目录，..表示安装私服的上级目录就是仓库的位置

runtime=${bundleBasedir}/nexus/WEB-INF  # nexus运行程序目录
```

 

访问：<http://localhost:8081/nexus/>

![img](./img/051.png) 

输入用户名和密码admin/admin123登录私服

 

### 4.4.3. 仓库类型

nexus 查看：

nexus 的仓库：

![img](./img/052.png) 

==7个仓库配置，4种类型==

- group(仓库组)： 一组仓库， 用来合并多个hosted/proxy仓库，通常我们配置自己的maven连接仓库组，用于从私服上下载jar包。


当使用仓库坐标的时候，按照组内顺序查找 

![img](./img/053.png) 

左侧配置表示仓库具有的仓库组。

![img](./img/054.png) 

- hosted(宿主) ：私服


==部署自己的 jar 到这个类型的仓库==，包括 releases 和 snapshot 两部分， Releases 公司内部发布版本仓库、 Snapshots 公司内部测试版本仓库

![img](./img/055.png) 

​	Releases ： 指公司内部已经发布正式版本的项目 （自己开发）

​	Snapshorts ： 指公司内部项目还处于测试版本的项目 （自己开发）

​	3rd party ： 私服配置指向第三方仓库 （其它公司开发 ）

- proxy(代理) ：私服

  ==代理仓库， 用于代理远程的公共仓库==，如 maven 中央仓库，用户连接私服，私服自动去中央仓库下载 jar 包或者插件。

![img](./img/056.png) 

通过私服去连接 apache或者 中央仓库（Central）

- virtual(虚拟)  兼容maven1格式的 jar 或者插件（已经不用了）


![img](./img/057.png) 

 

nexus 仓库默认在 sonatype-work 目录中：

![img](./img/058.png) 

​	· ==central：代理仓库，代理中央仓库，存放中央仓库下载的jar包==

![img](./img/059.png) 

​	· apache-snapshots：代理仓库

 	存储 snapshots 构件，代理地址 <https://repository.apache.org/snapshots/>

· central-m1： virtual 类型仓库， 兼容 Maven1 版本的 jar 或者插件

==· releases： 本地仓库，存储 releases （正式）构件。==

==· snapshots： 本地仓库，存储 snapshots （测试）构件。==

==· thirdparty：第三方仓库==

==· public：仓库组，用于指定下载jar包的仓库顺序==

## 4.5. 私服安装后配置（了解）

【路径】

1：index仓库索引（从中央仓库更新索引，用于下载）

2：storage存放仓库（存放jar包）

3：重新运行，方能生效

【讲解】

在运行私服后，自动在 nexus-work=${bundleBasedir}/../sonatype-work/nexus 目录

创建私服目录结构

![img](./img/060.png) 

### 4.5.1. 【第一步】indexr 仓库索引 （搜索坐标 就是搜索索引 ）

index，表示索引

![img](./img/061.png) 

storage，仓库（用于存放真正的jar包）

![img](./img/062.png) 

默认没有任何索引

![img](./img/063.png) 

索引下载地址：

下载地址：http://repo.maven.apache.org/maven2/.index/

下载如下文件：

![img](./img/064.png) 

从网上下载：indexer-cli-5.1.0.jar

![img](./img/065.png) 

将这三者放在一个文件夹，cmd执行：

![img](./img/066.png) 

 

Cmd命令：java -jar indexer-cli-5.1.0.jar -u nexus-maven-repository-index.gz -d indexer

比较慢，需耐心等待

![img](./img/067.png) 

会自动在本目录生成索引文件目录，如下图：

![img](./img/068.png) 

打开：

![img](./img/069.png) 

将生成的数据放置到D:\software\nexus-2.12.0-01-bundle\sonatype-work==\nexus\indexer\central-ctx==

注意：先删除之前的文件，再复制粘贴。

![img](./img/070.png) 

### 4.5.2. 【第二步】storage 存放jar包 

默认中央仓库没有jar包， 将本地仓库D:\apache-maven-3.1.1\repository中的内容覆盖 ==storage/central==

### 4.5.3. 【第三步】重新运行restart私服（nexus.bat start）

然后访问页面，你就发现，你可以使用私服了

![img](./img/071.png) 

索引：

![img](./img/072.png) 

仓库：

 ![img](./img/073.png)

【小结】

了解：私服的下载、安装、配置（公司搭建好）

## 4.6. 将项目发布到私服

【路径】

1：需求场景

2：配置

3：测试

### 4.6.1.  需求

​	企业中多个团队协作开发通常会将一些公用的组件、开发模块等发布到私服供其它团队或模块开发人员使用。
​	本例子假设多团队分别开发 ssm_model、ssm_dao、 ssm_service、 ssm_web，某个团队开发完在ssm_dao 会将 ssm_dao 发布到私服供 ssm_service 团队使用，==本例子会将 ssm_dao 工程打成==
==jar 包发布到私服。==

![img](./img/074.png) 

### 4.6.2. 配置

第一步： 需要在客户端即部署 ssm_dao 工程的电脑上配置 maven环境，并修改 D:\apache-maven-3.3.9\conf\settings.xml文件， 配置连接私服的用户和密码 。
​	此用户名和密码用于私服校验，因为私服需要知道上传的账号和密码是否和私服中的账号和密码一致。

```xml
<server>
      <id>releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
	<server>
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
```

其中

releases 连接发布版本项目仓库

snapshots 连接测试版本项目仓库

![img](./img/075.png) 

第二步： 配置项目 pom.xml
​	配置私服仓库的地址，本公司的自己的 jar 包会上传到私服的宿主仓库，根据工程的版本号决定上传到哪个宿主仓库，如果版本为 release 则上传到私服的 release 仓库，如果版本为snapshot 则上传到私服的 snapshot 仓库

​	将下面的代码，放置到ssm_parent中（与<dependencies>标签并列）

```xml
<!--工程中配置链接-->
 <distributionManagement>
  	<repository>
  		<id>releases</id><!-- 需要与私服releases中配置的id一致 -->
  		<name>itcast-releases</name>
  		<url>http://localhost:8081/nexus/content/repositories/releases/</url>	
  	</repository>
  	<snapshotRepository>
		<id>snapshots</id><!-- 需要与私服snapshots中配置的id一致 -->  	
  		<name>itcast-snapshots</name>
  		<url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
  	</snapshotRepository>
  </distributionManagement>
```

注意： pom.xml 这里< id > 和 settings.xml 配置 < id > 对应！

### 4.6.3. 测试

将项目 dao 工程打成 jar 包发布到私服：

1、 首先启动 nexus

2、 对 ssm_dao 工程执行 deploy 命令

![img](./img/076.png) 

控制台：

![img](./img/077.png) 

根据本项目pom.xml中version定义决定发布到哪个仓库，如果version定义为snapshot，执行 deploy后查看 nexus 的 snapshot仓库， 如果 version定义为 release则项目将发布到 nexus的 release 仓库，本项目将发布到 snapshot 仓库：

![img](./img/078.png) 

也可以通过 http 方式查看:

![img](./img/079.png) 

注意：如果使用maven的tomcat插件的时候，ssm_web启动tomcat，执行tomcat7:run，会抛出异常

![img](./img/080.png) 

[ERROR] Failed to execute goal on project ssm_dao: Could not resolve dependencies for project com.itheima:ssm_dao:jar:1.0-SNAPSHOT: Could not find artifact com.itheima:ssm_service:jar:1.0-SNAPSHOT -> [Help 1]

[ERROR] 

[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.

==解决方案：这里需要先将ssm_parent安装到本地仓库，异常解决。==

![img](./img/081.png) 

查看仓库信息

![img](./img/082.png) 

但是如果删除ssm_dao则程序不能运行，可否从私服上下载呢？能。

 

## 4.7. 从私服下载 jar 包

【路径】

1：需求场景

2：管理仓库组

3：配置

4：测试

### 4.7.1. 需求

​	没有配置 nexus 之前，如果本地仓库没有，去中央仓库下载，通常在企业中会在局域网内部署一台私服服务器， 有了私服本地项目首先去本地仓库找 jar，如果没有找到则连接私服从私服下载 jar 包，如果私服没有 jar 包私服同时作为代理服务器从中央仓库下载 jar 包，这样做的好处是一方面由私服对公司项目的依赖 jar 包统一管理，一方面提高下载速度， 项目连接私服下载 jar 包的速度要比项目连接中央仓库的速度快的多。
​	==本例子测试从私服下载 ssm_dao 工程 jar 包。==

### 4.7.2. 管理仓库组（用于下载）

​	nexus中包括很多仓库， hosted中存放的是企业自己发布的jar包及第三方公司的jar包，proxy 中存放的是中央仓库的 jar，为了方便从私服下载 jar 包可以将多个仓库组成一个仓库组，每个工程需要连接私服的仓库组下载 jar 包。
​	打开 nexus 配置仓库组，如下图：

![img](./img/083.png) 

上图中仓库组包括了本地仓库、代理仓库等。

### 4.7.3.  在 setting.xml 中配置私服仓库

​	在客户端的 settings.xml 中配置私服的仓库，由于 settings.xml 中没有 repositories 的配置标签需要使用 profile 定义仓库。

```xml
	<!-- 下载jar包配置 -->
	<profile> 
		<!--profile的id -->
		<id>dev</id>
		<repositories>
			<repository> <!--仓库id，repositories可以配置多个仓库，保证id不重复 -->
				<id>nexus</id> <!--仓库地址，即nexus仓库组的地址 -->
				<url>http://localhost:8081/nexus/content/groups/public/</url> <!--是否下载releases构件 -->
				<releases>
					<enabled>true</enabled>
				</releases> <!--是否下载snapshots构件 -->
				<snapshots>
					<enabled>true</enabled>
				</snapshots>
			</repository>
		</repositories>
		<pluginRepositories> <!-- 插件仓库，maven的运行依赖插件，也需要从私服下载插件 -->
			<pluginRepository> <!-- 插件仓库的id不允许重复，如果重复后边配置会覆盖前边 -->
				<id>public</id>
				<name>Public Repositories</name>
				<url>http://localhost:8081/nexus/content/groups/public/</url>
			</pluginRepository>
		</pluginRepositories>
	</profile>
```

使用 profile 定义仓库需要激活才可生效。

```xml
  <activeProfiles>
	<activeProfile>dev</activeProfile>
  </activeProfiles>
```

 

### 4.7.4.  测试从私服下载 jar 包

​	测试 1：局域网环境或本地网络即可
​	1：在 ssm-service 工程中添加以上配置后，添加 ssm-dao 工程的依赖，并删除本地仓库中 ssm_dao的曾经部署的工程。

![img](./img/084.png) 

2：项目先从本地仓库找 ssm_dao，找不到从私服找，由于之前执行 deploy 将 ssm_dao 部署到私服中，所以成功从私服下载 ssm_dao ，并在本地仓库保存一份。

执行ssm_web工程中的，tomcat7:run

![img](./img/085.png) 

3：看到控制台的信息：

![img](./img/086.png) 

4：看到本地仓库的信息

![img](./img/087.png)
测试 2：需要互联网环境
​	在项目的 pom.xml 添加一个依赖，此依赖在本地仓库和私服都不存在， maven 会先从本地仓库找，本地仓库没有再从私服找，私服没有再去中央仓库下载， jar 包下载成功在私服、本地仓库分别存储一份。

​	在ssm_service中的pom.xml中添加：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.7</version>
</dependency>
```

查看：本地仓库：

![img](./img/088.png) 

查看：私服：

![img](./img/089.png) 

## 4.8. 把第三方 jar 包放入本地仓库或私服

【路径】

1：使用cmd命令第三方jar包导入本地仓库

2：使用cmd命令第三方jar包导入私服

3：参数说明

4：使用私服管理控制台将第三方jar包存放到私服

【需求】

将fastjson-1.1.37包放在F:\a盘根路径 

### 4.8.1.  导入本地仓库（回顾）

随便找一个 jar 包测试， 可以先 CMD进入到 jar 包所在位置，运行

```java
	mvn install:install-file -DgroupId=com.alibaba -DartifactId=fastjsonm -Dversion=1.1.37 -Dfile=fastjson-1.1.37.jar -Dpackaging=jar
```

![img](./img/090.png) 

查看本地仓库

![img](./img/091.png) 

### 4.8.2.  导入私服

需要在 maven 软件的核心配置文件 settings.xml 中配置第三方仓库的 server 信息

```xml
	<server>
		<id>thirdparty</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
```

才能执行以下命令

```java
	mvn deploy:deploy-file -DgroupId=com.alimama -DartifactId=fastjsonmm -Dversion=1.1.37 -Dpackaging=jar -Dfile=F:/a/fastjson-1.1.37.jar -Durl=http://localhost:8081/nexus/content/repositories/thirdparty/ -DrepositoryId=thirdparty
```

![img](./img/092.png) 

查看私服：

![img](./img/093.png) 

查看私服上的文件：

![img](./img/094.png) 

### 4.8.3. 参数说明

![img](./img/095.png) 

DgroupId 和 DartifactId 构成了该 jar 包在 pom.xml 的坐标，项目就是依靠这两个属性定位。
自己起名字也行。
Dfile 表示需要上传的 jar 包的绝对路径。
Durl 私服上仓库的位置，打开 nexus——>repositories 菜单，可以看到该路径。
DrepositoryId 服务器的表示 id，在 nexus 的 configuration 可以看到。

![img](./img/096.png)
Dversion 表示版本信息，
关于 jar 包准确的版本：
包的名字上一般会带版本号，如果没有，可以解压该包，会发现一个叫 MANIFEST.MF 的文件，这个文件就有描述该包的版本信息。

![img](./img/097.png)
打开MANIFEST.MF文件。

![img](./img/098.png)

### 4.8.4. 将第三方jar包放置到私服 （操作二）

【需求】如何将classes12.jar上传到私服中的第三方仓库

第一步：选择3rd party，会看到下方的Artifact Upload

![img](./img/099.png) 

 

第二步：完成配置：

![img](./img/100.png) 

第三步：点击Add Artifact，最后点击upload Artifact（s）

![img](./img/101.png) 

第四步：查看上传的jar包

![img](./img/102.png) 

第五步：查看上传的classes.jar包的坐标：

![img](./img/103.png) 

测试从私服上下载。

 

## 4.9. 配置阿里私服或者开源中国私服

配置私服来加快maven下载速度。

Settings.xml



百度：开源中国maven。

```xml
  <mirrors>    
		<mirror>
			<id>nexus-osc</id>
			<mirrorOf>*</mirrorOf>
			<name>Nexus osc</name>
			<url>http://maven.oschina.net/content/groups/public</url>
		</mirror>
  </mirrors>
```

阿里云

```xml
    <mirror> 
		<id>nexus-aliyun</id>  
		<mirrorOf>central</mirrorOf>  
		<name>Nexus aliyun</name>  
		<url>http://maven.aliyun.com/nexus/content/groups/public</url>  
	</mirror>
```

缺点：私服中的jar包不全，或者jar下载的不完整，下载可能会有问题，因为可能产生.lastUpdate文件，此时不能中央仓库下载，导致下载的jar有问题。

 

# 5. 【补充】当下载jar的时候，如果断网，或者连接超时的时候，会自动在文件夹中创建一个名为\*.lastupdate的文件，当有了这个文件之后，当你再次联网的时候，那么该文件不会再自动联网，必须手动删除，才能正常下载使用。如果删除呢？

使用工具：

![img](./img/104.png) 

 

如何使用？打开工具

![img](./img/105.png) 

双击运行

![img](./img/106.png) 

 

Maven项目，如果正在下载一个jar，但是突然断网，此时，就会产生一个m2e-lastUpdated.，别指望下次上网就会自动下载，必须手动删除该文件，然后再进行下载。