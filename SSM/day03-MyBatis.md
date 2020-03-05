---
typora-copy-images-to: img
---

# day03-MyBatis 

# 学习目标

- [ ] 掌握Mybatis的延迟加载
- [ ] 掌握Mybatis缓存
- [ ] 掌握Mybatis注解开发

# 第一章-Mybatis 延迟加载策略 

## 知识点-Mybatis 延迟加载策略 

### 1.目标

​	通过前面的学习，我们已经掌握了 Mybatis 中一对一，一对多，多对多关系的配置及实现，可以实现对象的关联查询。实际开发过程中很多时候我们并不需要总是在加载一方信息时就一定要加载另外一方的信息。 此时就是我们所说的延迟加载。

### 2.路径

​	1.什么是延迟加载

​	2.一对一延迟加载实现

​    3.一对多延迟加载实现

### 3.讲解

#### 3.1什么是延迟加载 

​	延迟加载：就是在需要用到数据时才进行加载(查询)，不需要用到数据时就不加载数据。延迟加载也称懒加载.

​	坏处： 因为只有当需要用到数据时，才会进行数据库查询，这样在大批量数据查询时，因为查询工作也要消耗时间，所以可能造成用户等待时间变长，造成用户体验下降。 

​	好处： 先从单表查询，需要时再从关联表去关联查询，大大提高数据库性能，因为查询单表要比关联查询多张表速度要快.

#### 3.2使用 Assocation 实现延迟加载 (多对一,一对一)

##### 3.2.1需求

​	查询账户(Account)信息并且关联查询用户(User)信息。



​	先查询账户(Account)信息，当我们需要用到用户(User)信息时再查询用户(User)信息。

```sql
-- 1. 查询账户
SELECT * FROM t_account
-- 2, 再查询用户(等使用到用户再查询 account.getUser().getName())
--    再根据查询结果里面的uid查询当前账户所属的用户
SELECT * FROM t_user WHERE uid = 1
```

​	`sql:select * from account `

##### 3.2.2实现

+ User.java

```java
public class User{
    private int uid;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址
  
}
```

+ Account.java

```java
public class Account {
    private Integer aid;
    private Integer uid;
    private Double money;

    //表达关系:1个用户对应1个账户
    private User user;

}
```

+ AccountDao.java

```java
public interface AccountDao {
    /**
     * 查询账户信息(包含用户信息)
     *
     * @return
     */
    List<Account> findAccountList();

}
```

+ AccountDao.xml

```xml
<mapper namespace="com.itheima.dao.AccountDao">
    <select id="findAccountList"  resultMap="accountListId">
        SELECT * FROM t_account
    </select>
    <resultMap id="accountListId" type="Account">
        <id property="aid" column="aid"/>
        <result property="money" column="money"/>
        <result property="uid" column="uid"/>
        <!--association用于关联加载的对象,
                property属性:多方类里面一方对象的属性名
               javaType属性:一方对象的类型
               select属性: 用于关联查询另外的一方(User)对应的id
               column属性: 填写我们要传递给 select 映射的参数
               fetchType属性：lazy延迟
         -->
        <association property="user" javaType="User" select="com.itheima.dao.UserDao.findByUid" column="uid" fetchType="lazy" >
        </association>
    </resultMap>
</mapper>
```

+ UserDao.java

```java
public interface UserDao {
    /**
     * 根据id查询user
     * @param uid
     * @return
     */
    User findByUid(Integer uid);

}
```

+ UserDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.UserDao">
    <select id="findByUid"  resultType="User" parameterType="Integer">
        SELECT * FROM t_user WHERE  uid = #{uid}
    </select>
</mapper>
```

+ 测试

```java
public class DbTest {
    @Test
    public void fun01() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        AccountDao accountDao = sqlSession.getMapper(AccountDao.class);
        List<Account> list = accountDao.findAccountList();

        for (Account account : list) {
            User user = account.getUser();
            System.out.println(user);
        }
        sqlSession.close();
    }

}
```

##### 3.2.3小结

1. 拆分sql

   ```sql
   --先查询account数据
   select * from t_account;
   --当执行getUser的时候传入id执行sql查询出用户
   select * from user where id = 1;
   ```

  2.定义resultmap

```xml
  <resultMap id="AccountUserMap" type="Account">
        <id column="aid" property="aid"/>
        <result column="money" property="money"/>
        <result column="uid" property="uid"/>
        <!--
            当设置为fetchType=lazy，代表开启懒加载，如果是eager就是立刻加载
            当执行account的getUser方法的时候，执行select对应的sql语句，并把
            column对应的列的值作为参数传递过去，
            执行的结果封装为javaType指定的类型
            赋值给type对应类型的property指定的属性
        -->
        <association property="user" javaType="User"
                     select="com.itheima.dao.UserMapper.findById" column="uid" fetchType="lazy"/>
    </resultMap>
```

​	3.外部xml映射文件中的sql节点

```xml
<mapper namespace="com.itheima.dao.UserMapper">
    <select id="findById" resultType="User">
        select * from user where id = #{asd};
    </select>
</mapper>
```

 	4.外部xml映射文件对应的接口

```java
//第二步：再根据查询出来的用户id去查询account
public interface AccountMapper {
    public List<Account> findAccountUsers();
}
```

​	

#### 3.3Collection 实现延迟加载  (一对多)

##### 3.3.1需求

​	完成加载用户对象时，查询该用户所拥有的账户信息。

​	 等账户信息使用的时候再查询.

```sql
-- 1.先把用户查询出来
SELECT * FROM t_user
-- 2. 等使用了该用户的账户的时候再把账户查询出来(user.getAccounts()...)
--    把查询出来的uid去查询当前用户的账户信息
SELECT * FROM t_account WHERE uid = 1
```

##### 3.3.2实现

+ Account.java

```java
public class Account {
    private Integer aid;
    private Integer uid;
    private Double money;
}
```

+ User.java

```java
public class User implements Serializable{
    private int uid;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址

    //表达关系:1个用户对应多个账户
    private List<Account> accounts;
}
```

+ UserDao.java

```java
public interface UserDao {
    /**
     * 查询所有的用户对应的账户信息
     * @return
     */
    List<User> findUserAccountList();

}
```

+ UserDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.UserDao">
    <select id="findUserAccountList" resultMap="userListId">
        SELECT * FROM t_user
    </select>
    <resultMap id="userListId" type="user">
        <result property="uid" column="uid"/>
        <result property="username" column="username"/>
        <result property="sex" column="sex"/>
        <result property="birthday" column="birthday"/>
        <result property="address" column="address"/>
        <!--加载多方数据,property属性:一方类里面List集合的属性名; ofType:List中的对象类型-->
        <collection property="accounts" ofType="account" select="com.itheima.dao.AccountDao.findByUid" column="uid" fetchType="lazy"/>
    </resultMap>
</mapper>
```

+ AccountDao.java

```java
public interface AccountDao {
    /**
     * 根据uid查询当前用户的账户
     * @return
     */
    List<Account> findByUid(Integer uid);

}
```

+ AccountDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.AccountDao">
    <select id="findByUid" resultType="Account" parameterType="Integer" >
        SELECT * FROM t_account WHERE  uid = #{uid}
    </select>
</mapper>
```

+ 测试方法

```java
public class DbTest {
    @Test
    public void fun01() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        List<User> list = userDao.findUserAccountList();

        for (User user : list) {
            System.out.println(user.getAccounts().size());
        }
        sqlSession.close();
    }

}
```

##### 3.3.3小结

1. 拆分sql

   ```sql
   --第一步，查询所有用户信息
   select * from user;
   --第二步,通过用户id查询账户信息
   select * from t_account where uid = 1;
   ```
   
2. xml

   ```xml
   <mapper namespace="com.itheima.dao.UserMapper">
       <resultMap id="userAccountMap" type="User">
           <id column="id" property="id"/>
           <result column="username" property="username"/>
           <result column="birthday" property="birthday"/>
           <result column="sex" property="sex"/>
           <result column="address" property="address"/>
           <!--
               collection:一对多
               property：对应user类的accounts属性名
               ofType：List<T> 就是要转化的T的类型=>private List<Account> accounts。就是Account
           -->
           <collection property="accounts" ofType="Account"
                       column="id" select="AccountMapper.findByUid" fetchType="lazy"
             />
       </resultMap>
   
       <select id="findUserAccounts" resultMap="userAccountMap">
           select * from user
       </select>
   </mapper>
   ```

   ```xml
   <mapper namespace="AccountMapper">
       <select id="findByUid" resultType="Account">
           select * from t_account where uid = #{id}
       </select>
   </mapper>
   ```

3. 接口

   ```java
   public interface UserMapper {
       public List<User> findAll();
   }
   public interface AccountMapper {
       public List<Account> findByUid(int uid);
   }
   ```


### 4.总结

| 类别   | 特点                        |
| ---- | ------------------------- |
| 立即加载 | 只要一调用方法，则马上发起查询           |
| 延迟加载 | 只有在真正使用时，才发起查询，如果不用，则不查询。 |

1.缺陷

​	会对数据库进行多次的io操作，浪费了系统资源，在真正开发的时候，往往都是使用关联查询，建立合适的索引

一次查询出数据，只查询需要的字段，不要select *



# 第二章-MyBatis缓存

## 知识点-缓存概述

### 1.目标

- [ ] 掌握MyBatis缓存类别

### 2.路径

1.什么是缓存（内存空间）

2.为什么适用缓存（降低数据库io频率，提升性能）

3.缓存的适用情况(数据修改不频繁使用，频繁就不使用)

4.mybatis缓存类别

### 3.讲解

#### 3.1缓存概述

​	缓存就是一块内存空间.保存临时数据

#### 3.2为什么使用缓存

​	将数据源（数据库或者文件）中的数据读取出来存放到缓存中，再次获取的时候 ,直接从缓存中获取，可以减少和数据库交互的次数 ,这样可以提升程序的性能！

#### 3.3缓存的适用情况

+ 适用于缓存的：经常查询但不经常修改的(eg: 省市,类别数据)，数据的正确与否对最终结果影响不大的，一些热点数据（访问量非常大的数据）
+ 不适用缓存的：经常改变的数据 , 敏感数据（例如：股市的牌价，银行的汇率，银行卡里面的钱)等等, 

#### 3.4MyBatis缓存类别

​	一级缓存：它是sqlSession对象的缓存，自带的(不需要配置的)不可卸载(不想用还不行)的.  一级缓存的生命周期与sqlSession一致。

​	二级缓存：它是SqlSessionFactory的缓存。只要是同一个SqlSessionFactory创建的SqlSession就解压共享二级缓存的内容，并且可以操作二级缓存。二级缓存如果要使用的话，需要我们自己手动开启(需要配置的)。

### 4.小结

1. 缓存：就是内存空间，一般都是使用map

2. 适用缓存的情况

   ​	1.数据不经常修改

   ​	2.数据对准确性要求不高

   ​	3.热点数据

   3.不适用缓存的情况

​		1.数据经常被修改，缓存就会经常失效，会频繁的查询数据库，并且缓存会占用内存空间，写入缓存也会造成系统资源消耗，这时候反而得不偿失。

​		2.对数据准确性要求高（比如用户的账号的钱）

   4.mybatis缓存类别

​		1.一级缓存sqlsession级别，sqlSession既可以使用自己的一级缓存也可以使用共享的二级缓存。

​		2.二级缓存SqlSessionFactory级别，他是所有的sqlSaession共享的缓存空间

## 知识点-一级缓存

### 1.目标

- [ ] 掌握MyBatis一级缓存

### 2.路径

1. 证明一级缓存的存在（SqlSession级别）
2. 了解缓存过程
3. 清除一级缓存

### 3.讲解

#### 3.1证明一级缓存的存在

```java
    //证明一级缓存的存在
    public void fun01() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        User user01 = userDao.findByUid(1);
        User user02 = userDao.findByUid(1);

        System.out.println(user01 == user02);

        sqlSession.close();
    }
```

#### 3.2 一级缓存分析

​	![image-20191231155557055](img\image-20191231155557056.png)

​	![img](img/tu_2.png)

​	第一次发起查询用户 id 为 1 的用户信息，先去找缓存中是否有 id 为 1 的用户信息，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。第二次发起查询用户 id 为 1 的用户信息，先去找缓存中是否有 id 为 1 的用户信息，缓存中有，直接从缓存中获取用户信息。 

​	如果 sqlSession 去执行 commit操作（执行插入、更新、删除），清空 SqlSession 中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

#### 3.3测试一级缓存清空

+ 关闭session

```java
    @Test
    public void fun01() throws IOException {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        User user1 = userDao.findByUid(1);
        User user2 = userDao.findByUid(1);

        System.out.println(user1 == user2); //true

        sqlSession.close();

        SqlSession sqlSession02 = SqlSessionFactoryUtil.openSession();
        UserDao userDao02 = sqlSession02.getMapper(UserDao.class);

        User user3 = userDao02.findByUid(1);//再次从数据库查询了

    }
```

+ 更新数据也会清空一级缓存

```java
    @Test
    public void fun02() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);

        User user01 = userDao.findByUid(1);
        user01.setSex("女");
        userDao.update(user01);
        sqlSession.commit();

        User user02 = userDao.findByUid(1);

        System.out.println(user01 == user02);

        sqlSession.close();
    }
```

### 4.小结

1. 一级缓存默认开启（sqlsession级别）

2. 它的本质其实是个hashMap

   ![image-20191231160222934](img\image-20191231160222935.png)

3. 清除缓存方式，sqlSession.clearCache()，或者是sqlSession.close()直接关闭,进行增删改sqlSession.commit()也会清除。

   **只要发生了可能造成数据改变的行为就会清空一级缓存**



## 知识点-二级缓存

### 1.目标

- [ ] 掌握MyBatis二级缓存

### 2.路径

1. 二级缓存的结构
2. 二级缓存的使用
3. 二级缓存测试

### 3.讲解

​	二级缓存是SqlSessionFactory的缓存。只要是同一个SqlSessionFactory创建的SqlSession就共享二级缓存的内容，并且可以操作二级缓存.

#### 3.1二级缓存的结构

![img](img/tu_3.png)

​			**应该是sqlSession提交或者关闭的时候才会写入二级缓存**

#### 3.2二级缓存的使用

步骤:

1. 先在核心配置文件中配置一下 cacheEnabled true
2. 在sql映射文件中写入<cache/>开启该namespace下的缓存
3. 在statement就是xml映射文件中的sql查询节点上写 useCache='true'

+ 在 SqlMapConfig.xml 文件开启二级缓存 

![img](img/tu_4.png)

​	==因为 cacheEnabled 的取值默认就为 true==，所以这一步可以省略不配置。为 true 代表开启二级缓存；为 false 代表不开启二级缓存。  

+ 配置相关的 Mapper 映射文件 

  `<cache>` 标签表示当前这个 mapper 映射将使用二级缓存，区分的标准就看 mapper 的 namespace 值。 

  ![img](img/tu_5.png)

+ 配置 statement 上面的 useCache 属性 

  ​	将 UserDao.xml 映射文件中的 `<select>`标签中设置 useCache=”true”代表当前这个 statement 要使用二
  级缓存，如果不使用二级缓存可以设置为 false。
  ​	注意： 针对每次查询都需要最新的数据 sql，要设置成 useCache=false，禁用二级缓存。 

  ![img](img/tu_6.png)

#### 3.3测试

```java
    @Test
    //证明二级缓存
    public void fun02() throws Exception {
        SqlSession sqlSession01 = SqlSessionFactoryUtils.openSession();
        UserDao userDao = sqlSession01.getMapper(UserDao.class);
        User user01 = userDao.findByUid(1);
        sqlSession01.close();

        SqlSession sqlSession02 = SqlSessionFactoryUtils.openSession();
        UserDao userDao02 = sqlSession02.getMapper(UserDao.class);
        User user02 = userDao02.findByUid(1);//没有查询数据库了,直接从二级缓存获得

        sqlSession01.close();
    }
```

​	经过上面的测试，我们发现执行了两次查询，并且在执行第一次查询后，我们关闭了一级缓存，再去执行第二次查询时，我们发现并没有对数据库发出 sql 语句，所以此时的数据就只能是来自于我们所说的二级缓存。 

### 4.小结

#### 4.1二级缓存

​		1.全局配置文件

````xml
<settings>
        <!--可以不写，默认是true-->
        <setting name="cacheEnabled" value="true"/>
    </settings>
````

​		2.sql映射文件

````xml
<!--
1.代表在sqlSessionFactory这个二级缓存空间开启了当前namespace的缓存区域
如果配置readOnly="true"可以避免序列化，如果不配，缓存的对象实现序列化接口
-->
<cache/>
<!--
2.代表当前查询使用缓存
-->
<select usercache="true" />
````

​		3.对象

````java
//如果<cache/>没有配置readOnly="true"，必须实现序列化接口
@Data
public class User implements Serializable {
    private Integer id;//int(11)
    private String username;//varchar(32
    private Date birthday;// datetime',
    private String sex; //char(1)
    private String address;//varchar(256)
}
````

​		4.二级缓存在哪

​	在sqlSessionFactory里面

![image-20191231164859506](img\image-20191231164859506.png)

​	



# 第三章-MyBatis注解开发

​	这几年来注解开发越来越流行， Mybatis 也可以使用注解开发方式，这样我们就可以减少编写 Mapper映射文件了。 本次我们先围绕一些基本的 CRUD来学习，再学习复杂映射关系及延迟加载。

## 知识点-使用 Mybatis 注解实现基本CRUD 

### 1.目标

- [ ] 使用 Mybatis 注解实现基本CRUD

### 2.分析

​	1.查询 @Select("sql语句")

​	2.新增 @Insert("sql语句")

​		返回key键@SelectKey("")

​	3.删除 @Delete()

​	4.修改 @Update()

### 3.实现

+ Dao

```java
public interface UserDao {


@Select("SELECT * FROM t_user")
List<User> findAll();


@SelectKey(keyProperty = "uid",resultType = int.class,before = false,statement="SELECT LAST_INSERT_ID()")
@Insert("INSERT INTO t_user(username,sex,birthday,address)VALUES(#{username},#{sex},#{birthday},#{address})")
void save(User user);


@Update("UPDATE t_user SET  username = #{username} ,sex = #{sex}, birthday = #{birthday},address = #{address} WHERE uid = #{uid}")
void update(User user);


@Delete("DELETE  FROM t_user WHERE uid = #{uid}")
void delete(int uid);


@Select("SELECT  * FROM  t_user WHERE  uid = #{uid}")
User findByUid(int uid);


@Select("SELECT * FROM t_user WHERE username like #{firstName}")
List<User> findByFirstName(String firstName);


}
```

+ SqlMapConfig.xml

 ![1535361902219](img/1535361902219.png)

### 4.小结

```java
@select代替<select/>
@insert代替<insert/>
	@selectKey代替<selectKey>
		<selectKey:order（after,before）
		@selectKey:before(true,false)
@delete代替<delete
@update代替<update
```

## 知识点-使用Mybatis注解实现复杂关系映射开发

### 1.目标

- [ ] 掌握Mybatis注解开发

### 2.路径

1. 复杂关系映射的注解说明
2. 一对一关系注解开发延迟加载
3. 一对多关系的注解开发和延迟加载

### 3.讲解 

​	实现复杂关系映射之前我们可以在映射文件中通过配置<resultMap>来实现，但通过后我们发现并没有@ResultMap 这个注解。下面我们一起来学习@Results 注解， @Result 注解， @One 注解， @Many注解。 

#### 3.1复杂关系映射的注解说明 

+ @Results 注解 , 代替的是标签<resultMap> 

```java
//该注解中可以使用单个@Result 注解，也可以使用@Result 集合
@Results（{@Result（）， @Result（） }）或@Results（@Result（））
```

+ @Result 注解 ,代替了 <id>标签和<result>标签 ,

```java
@Result(column="列名",property="属性名",one=@One(select="指定用来多表查询的 sqlmapper"),many=@Many(select=""))

@Resutl 注解属性说明	
    column 数据库的列名
    Property 需要装配的属性名
    one 需要使用的@One 注解（@Result（one=@One）（）））
    many 需要使用的@Many 注解（@Result（many=@many）（）））
```

+ @One 注解（一对一）,代替了<assocation>标签，是多表查询的关键，在注解中用来指定子查询返回单一对象。 

```java
@Result(column="列名",property="属性名",one=@One(select="指定用来多表查询的 sqlmapper"))
```

+ @Many 注解（一对多） ,代替了<Collection>标签,是是多表查询的关键，在注解中用来指定子查询返回对象集合 

  ​	注意：聚集元素用来处理“一对多”的关系。需要指定映射的 Java 实体类的属性，属性的 javaType（一般
  为 ArrayList） 但是注解中可以不定义； 

```java
@Result(property="",column="",many=@Many(select=""))
```

#### 3.2使用注解实现一对一复杂关系映射及延迟加载 

##### 3.2.1需求

​	查询账户(Account)信息并且关联查询用户(User)信息。

​		1.先去查询出所有的account。

​		2.根据accountid查询出对应的user

​	先查询账户(Account)信息，当我们需要用到用户(User)信息时再查询用户(User)信息。

##### 3.2.2实现

+ User.java

```java
public class User implements Serializable{
    private int uid;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址
 }
```

- Account.java

```java
public class Account {
    private Integer aid;
    private Integer uid;
    private Double money;

    //表达关系:1个用户对应1个账户
    private User user;
}
```

- AccountDao.java

![img](img/tu_9.png)

```java
public interface AccountDao {
    /**
     * 查询账户信息(包含用户信息)
     * @return
     */
    @Select("SELECT * FROM t_account")
    @Results(value = {
            @Result(column = "aid",property = "aid",id = true),
            @Result(column = "uid",property = "uid"),
            @Result(column = "money",property = "money"),
            @Result(column = "uid",property = "user",one=@One(select="com.itheima.dao.UserDao.findByUid",fetchType = FetchType.LAZY))
    })
    List<Account> findAccountList();

}
```

+ UserDao.java

```java
public interface UserDao {
    /**
     * 根据id查询user
     * @param uid
     * @return
     */
    @Select("SELECT * FROM t_user WHERE  uid = #{uid}")
    User findByUid(Integer uid);

}
```

+ 测试

```java
public class DbTest {
    @Test
    public void fun01() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        AccountDao accountDao = sqlSession.getMapper(AccountDao.class);
        List<Account> list = accountDao.findAccountList();

        for (Account account : list) {
            User user = account.getUser();
            System.out.println(user);
        }
        sqlSession.close();
    }
}
```

##### 3.2.3小结

![1561971242935](D:\讲义\ssm框架\day03\笔记\img\1561971242935.png)

#### 3.3使用注解实现一对多复杂关系映射及延迟加载 

##### 3.3.1需求

​	完成加载用户对象时，查询该用户所拥有的账户信息。 

​	等账户信息使用的时候再查询.

##### 3.3.2实现

+ User.java

```java
public class User {
    private Integer uid;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址

    //用于保存用户的多个账户信息
    private List<Account> accounts;
    
	//...    
}
```

- UserDao.java

![img](img/tu_10.png)

```java
public interface UserDao {
    /**
     * 查询所有的用户对应的账户信息
     * @return
     */
    @Select("SELECT * FROM t_user")
    @Results(value = {
            @Result(column = "uid",property = "uid",id = true),
            @Result(column = "username",property = "username"),
            @Result(column = "sex",property = "sex"),
            @Result(column = "birthday",property = "birthday"),
            @Result(column = "address",property = "address"),
            @Result(column = "uid",property ="accounts",many = @Many(
                    select = "com.itheima.dao.AccountDao.findByUid",fetchType = FetchType.LAZY
            ))
    })
    List<User> findUserAccountList();

}
```

+ AccountDao.java

```java
public interface AccountDao {

    /**
     * 根据uid查询当前用户的账户
     *
     * @return
     */
    @Select("SELECT * FROM t_account WHERE  uid = #{uid}")
    List<Account> findByUid(Integer uid);

}
```

+ 测试

```java
public class DbTest {
    @Test
    public void fun01() throws Exception {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        List<User> list = userDao.findUserAccountList();

        for (User user : list) {
            System.out.println(user.getAccounts().size());
        }
        sqlSession.close();
    }

}
```

##### 3.3.3小结

1. Results 代替resultMap, @Result  代替id【配置id属性=true】,result,collection【配置many属性, many属性是@Many(select="" ,fetchType="")】

2. 图![1561928442198](D:\讲义\ssm框架\day03\笔记\img\1561928442198.png) 

