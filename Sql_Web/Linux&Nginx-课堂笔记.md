# Linux&Nginx-课堂笔记



## 一、在Linux里安装软件

### 1. rpm命令

- rpm: redhat package manager，红帽软件包管理套件。不需要联网就能使用，有时候依赖比较多，会麻烦
- 常用命令：
  - 安装：`rpm -ivh 软件包`
    - `i` ：安装模式
    - `v` ：显示执行过程
    - `h` ：安装时列出hash标记
  - 查询：`rpm -qa`
    - `q` ：使用查询模式
    - `a` ：查询所有软件
  - 卸载：`rpm -e --nodeps 软件包`
    - `e` ：erase, 删除模式
    - `nodeps` ：不验证软件之间的相互关联性  no dependency

### 2. yum命令

- yum：yellow dog updater、modifed，是红帽基于rpm提供的软件包管理器，能够从指定服务器自动下载并安装软件，可以自动处理软件的依赖。
- 注意：使用yum命令需要联网
- 常用命令：
  - 安装：`yum install 软件名`
  - 卸载：`yum remove 软件名`

### 3. 部署一个web项目到Linux里

#### 步骤

* 把web应用，被idea编译之后的结果拷贝出来----可以直接部署的
  * 把`curd`项目，idea编译后的文件夹，拷贝出来
* 把web应用上传到Linux里，放在tomcat的webapps文件夹里
  * 把文件夹上传到Linux
* linux里的数据库初始化
  * 把`curd`练习里数据库脚本，在Linux的数据库里执行一遍
* 把Tomcat启动
* 在自己电脑上访问:`http://192.168.59.128:8080/curd`

#### 注意

* 在Windows里开发的web项目。Windows里使用的JDK是1.8的==开发/编译的版本是1.8
* 在Linux里运行开发好的web项目。Linux里使用的JDK是1.8的===运行环境的版本是1.8
* 要求：运行环境 jdk版本 >= 开发/编译版本，否则不能运行
* 后边要学习Maven，可以指定编译的版本

## 二、Nginx

### 1. Nginx简介

#### 目标

* 了解Nginx的三大作用
* 理解负载均衡的概念
* 理解反向代理的概念

#### 讲解

##### 1.1 什么是Nginx

​	Nginx（engine x）是一款[轻量级]的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。也就是说，我们在Nginx上可以：

1. 可以发布静态网站（作为web服务器）
2. 可以实现**负载均衡**（通过**反向代理**）
3. 可以作为邮件代理服务器实现收发邮件等功能

其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。



##### 1.2 负载均衡

​	所谓的负载均衡是指，当同时有多个用户访问我们的服务器的时候，为了减少服务器的压力，我们需要将用户引入**服务器集群**中 压力较轻的服务器，分担较重压力服务器的负担。

​	Nginx默认采用轮询的方式（挨个分配请求）

![25](img/25.png)



##### 1.3 反向代理

###### 正向代理

- 通常简称为代理。是**客户端的代理**(通常需要在客户端进行**配置**)
- 比如：翻墙软件

![image-20191216111137466](img/image-20191216111137466.png)

###### 反向代理

* 服务端的代理，跟客户端没有关系，通常用于实现服务器的负载均衡，和服务端的安全配置

* 负载均衡：代理服务器把客户端的请求，转发给压力较小的服务器来处理
* 安全配置：在代理服务器上可以增加防火墙、安全过滤
* 对外提供统一的访问地址：服务器集群里有多个服务器，但是对象的访问地址是代理服务器的地址，对外的地址只有一个

![image-20191216110931365](img/image-20191216110931365.png)

#### 小结

* Nginx作用：
  * 部署静态项目（应用服务器，作用和Tomcat类似）
  * 实现负载均衡（通过反向代理）
  * 作为邮件收发的代理服务器
* 负载均衡：把客户端的请求，交给服务器集群里  压力较小的服务器来处理
* 正向代理：客户端的代理，和服务端无关
* 反向代理：服务端的代理，和客户端无关
  * 服务器集群，对外访问地址只有一个（提供代理服务器的地址）
  * 实现负载均衡（由代理服务器负责，把请求交给压力小的服务器）
  * 提高安全性（代理服务器上可以增加一些安全措施）

### 2. 安装Nginx

#### 目标

* 在Windows里启动、关闭Nginx
* 在Linux里安装Nginx软件
* 在Linux里启动、关闭Nginx

#### 讲解

##### 2.1 Windows版Nginx

###### 2.1.1 下载安装Nginx

- 下载地址：<http://nginx.org/en/download.html>

- 得到压缩包《nginx-1.14.0-windows.zip》，免安装，直接解压即可

- 目录结构：

  ![28](img/28.png)

###### 2.1.2 Nginx基本使用方式

1. 打开cmd，切换到nginx所在的目录

   或者：在nginx文件夹里，按着shift点击鼠标右键，选择`在此处打开PowerShell窗口`

2. 启动nginx：`start .\nginx.exe`

   * 启动成功后，在任务管理器里会有两个名称为`nginx`的进程

   ![1568429018085](img/1568429018085.png)

   * 启动后，可以使用浏览器访问nginx：`http://localhost:80`

3. 重新载入配置文件：`.\nginx.exe -s reload`

4. 关闭nginx：` .\nginx.exe -s stop`

![image-20191216163619751](img/image-20191216163619751.png)

##### 2.2. Linux安装Nginx

###### 2.2.1 下载安装Linux版Nginx

- 参考安装文档《配置JavaEE运行环境(Linux).doc》

###### 2.2.2 Nginx基本使用方式

1. 切换到nginx的安装目录的`sbin`文件夹里

2. 启动nginx：`./nginx`

   启动后，可以使用浏览器访问nginx：`http://Linux的ip:80`

3. 重新载入配置文件：`./nginx -s reload`

4. 关闭nginx：`./nginx -s stop`

![image-20191216171909461](img/image-20191216171909461.png)

#### 小结

* windows版的nginx，基本使用：通过cmd/powerShell命令操作
  * 启动nginx：`start nginx.exe`
  * 重启nginx：`nginx.exe -s reload`
  * 关闭nginx：`nginx.exe -s stop`

### 3. 使用Nginx

#### 目标

* 能够使用Nginx部署静态项目
* 能够使用Nginx实现简单的负载均衡（通过反向代理）

#### 讲解

##### 3.1 部署静态项目

​	没有动态资源的web项目（比如：JSP、Servlet）

1. 准备一个静态项目

   ![image-20191216163937682](img/image-20191216163937682.png)
   
2. 打开Nginx的配置文件：`conf/nginx.conf`，修改如下内容：

   ```
   server{
   	listen       80;
       server_name  localhost;
       
       location / {
           root   静态项目路径，比如 E:\\travel;
           index  默认欢迎页面，比如 index.html;
       }    
   }
   ```

3. 启动nginx

4. 打开浏览器，访问nginx

##### 3.2 实现负载均衡

1. 准备多个服务器

   - 在多个服务器里部署web项目
   - 把多个服务器同时启动

2. 修改nginx配置文件nginx.conf

   ```
   upstream servers{  # servers是自定义的名称，用于配置多个服务器的地址
   	server 192.168.190.3:8080 weight=10;  #第1个服务器的ip和端口地址  weight：权重
   	server 192.168.190.4:80 weight=5;    #第2个服务器的ip和端口地址 weight：权重
   	...
   }
   server{
   	listen       80;
       server_name  localhost;	
       
       location /{
           proxy_pass http://servers;  # http://服务器池子的名称;
           proxy_set_header Host $Host; # 把客户端的请求头，转发给目标服务器
       	index  欢迎页面;
       }
   }
   ```

3. 启动nginx

4. 打开浏览器，访问nginx

#### 小结

* 使用nginx部署静态项目：

  1. 准备一个静态项目（注意：里边不要有动态资源，不要有中文、空格、特殊字符）
  2. 修改nginx的`conf/nginx.conf`配置文件，找到其中`http->server->location /`

  ```
  location /{
  	root 静态项目路径，如果路径里的反斜杠，要转义;
  	index 欢迎页面;
  }
  ```

  3. 启动/重启nginx，使用浏览器访问

* 使用nginx实现负载均衡

  1. 准备工作：准备多台服务器（真实环境是多台服务器电脑，目前以同一电脑里不同Tomcat演示）

     * 准备三个Tomcat，分别修改三个Tomcat的`conf/server.xml`中，端口的配置--端口不冲突
     * 三个Tomcat里部署相同的项目（现在使用的是自带的ROOT）
     * 启动三个Tomcat

  2. 配置Nginx：修改`conf/nginx.conf`

     ```
         upstream tomcats{
             server localhost:8081 weight=10;
             server localhost:8082 weight=5;
             server localhost:8083 weight=2;
         }
     
         server {
             listen       80;
             server_name  localhost;
     
             #charset koi8-r;
     
             #access_log  logs/host.access.log  main;
     
             location / {
                 proxy_pass http://tomcats;
                 proxy_set_header Host $Host;
             }
         	.......
         }
     ```

  3. 启动nginx，使用浏览器通过nginx访问
  
     http://localhost:80

## 三、JDK的动态代理

### 1. 什么是代理

#### 代理是什么

我们 ==> 房东.租房()

我们 ==> 中介.租房() ==> 房东.租房()

你们 ==> 明星.唱歌()

你们 ==> 经纪人.唱歌() ==> 明星.唱歌()



* 代理：当我们不能/不方便直接调用目标对象，可以通过代理对象，间接调用目标对象
* 作用：
  * 代理对象可以对目标对象进行功能增强（而目标对象不需要做任何修改）
  * 代理对象可以对目标对象进行控制
* 使用场景：
  * 事务管理：我们写代码不需要有任何事务的操作，由Spring框架对我们的代码进行功能增强：增加事务管理的代码
  * Spring框架中，AOP的本质就是动态代理：在不修改我们代码的情况下，对我们的代码进行功能增强

#### 代理技术的分类

* 静态代理：代理类是提前创建好的，我们直接创建代理对象，然后通过代理对象间接调用目标对象
* 动态代理：代理类不存在，我们的代码在运行过程中，动态生成代理类和代理对象，然后再通过代理对象间接调用目标对象

#### 动态代理的技术

* 基于接口的动态代理：JDK的动态代理 `Proxy.newProxyInstance()`
* 基于子类的动态代理：cglib

### 2. JDK的动态代理

#### API介绍

* JDK的动态代理是基于接口的
  * 其它动态代理技术：cglib的动态代理，可以没有接口，是基于子类的

```java
Object proxy = Proxy.newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h);
```

* loader：类加载器
* interfaces：接口的字节码对象数组
* h：InvocationHandler的实现类对象，用来写代理对象要做的事情。接口里的方法，在代理对象的方法被调用时会执行

#### 使用示例

* 有接口：`Star`

```java
public interface Star {
    void sing(String song);
    String dance(String dance);
}
```

* 有目标类：`SuperStar`

```java
public class SuperStar implements Star {
    @Override
    public void sing(String song) {
        System.out.println("明星唱：" + song);
    }

    @Override
    public String dance(String dance) {
        System.out.println("明星跳：" + dance);
        return "明星跳完了";
    }
}
```

* 通过JDK的API，得到目标类的代理对象。通过代理对象调用目标对象

```java
/**
 * 代理对象的测试类，调用代理对象
 */
public class ProxyTest {
    public static void main(String[] args) {
        //---------直接调用目标对象---------
        Star star = new SuperStar();
        star.sing("好汉歌");
        String dance = star.dance("霹雳舞");
        System.out.println(dance);

        System.out.println("-------------------------");


        //---------通过代理对象间接调用目标对象--------
        //得到一个代理对象：通过JDK的API，生成一个代理类，得到代理类的对象

        /**
         * Proxy.newProxyInstance()需要的参数：
         *  参数1: ClassLoader loader，类加载器
         *  参数2：Class[] interfaces，接口的字节码对象数组
         *      String[] strArr = new String[]{"a", "b", "c"}
         *      Class[]  clsArr = new Class[]{ Star.class }
         *  参数3：InvocationHandler h，是一个接口，通常是传递进来一个匿名内部类 对象
         *      接口里有一个方法invoke，这个方法，在我们每次调用代理对象的任意方法时，实际调用的就是这个invoke方法
         */
        Star proxy = (Star) Proxy.newProxyInstance(
                star.getClass().getClassLoader(),
                star.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     * 我们调用代理对象的任意方法，实际上被调用的就是这个方法
                     * @param proxy 代理对象
                     * @param method 我们所调用的方法的Method对象
                     * @param args   我们调用方法时，所传递的实参数组
                     * @return
                     * @throws Throwable
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //System.out.println("invoke....调用的方法是：" + method.getName() + ",  传递的实参是：" + Arrays.toString(args));

                        //调用之前，前置增强：彩排
                        System.out.println("前置增强：彩排");
                        //唱歌方法调用之前，修改了实参值
                        if ("sing".equals(method.getName())){
                            args[0] = "绿光";
                        }


                        //调用目标对象的方法。
                        //代理对象里可以不调用目标对象（意味着：就算没有目标对象，也可以得到一个代理对象，实现功能）所有功能都由代理对象来完成
                        Object result = method.invoke(star, args);

                        //调用之后，后置增强：收钱
                        System.out.println("后置增强：收钱");

                        //把目标对象方法执行的返回值，返回给调用者
                        return result;
                    }
                }
        );


        //调用代理对象的方法
        proxy.sing("好汉歌");
        String dance1 = proxy.dance("霹雳舞");
        System.out.println(dance1);
    }
}
```



# 复习

1. 安装jdk

2. 安装MySql

3. 安装Tomcat

4. nginx的作用

   1. 可以部署静态项目
   2. 可以实现负载均衡
   3. 可以作为邮件收发的代理服务器

5. nginx相关的概念

   1. 负载均衡：把客户端的请求，交给服务器集群里 压力小的服务器
   2. 正向代理：客户端的代理，和服务端无关。
   3. 反向代理：服务端的代理，和客户端无关
      1. 服务器集群对外访问地址，只有一个：代理服务器的地址
      2. 实现负载均衡
      3. 提高安全性

6. nginx的使用

   1. Windows版的：
      * 启动：`start nginx.exe`
      * 重启：`nginx.exe -s reload`
      * 关闭：`nginx.exe -s stop`
   2. Linux版的：
      * 启动：`./nginx`
      * 重启：`./nginx -s reload`
      * 关闭：`./nginx -s stop`

7. nginx部署静态项目

   1. 准备一个静态项目
   2. 修改nginx的`conf/nginx.conf`配置文件

   ```
   server{
   	...
   
   	location /{
   		root 静态项目的路径;
   		index 默认欢迎页面;
   	}
   }
   ```

8. nginx实现负载均衡

   1. 准备工作：准备多台服务器
   2. 修改nginx的`conf/nginx.conf`配置文件

   ```xml
   upstream tomcats{
   	server 服务器的ip:port weight=权重;
   	server 服务器的ip:port weight=权重;
   }
   server{
   	...
   	location /{
   		proxy_pass http://tomcats;
   		proxy_set_header Host $Host;
   	}
   }
   ```

9. JDK的动态代理

   * API语法

   ```java
   Object proxy = Proxy.newProxyInstance(
       ClassLoader loader,
       Class[] interfaces,
       InvocationHandler h
   );
   ```

   * loader：提供一个类加载器

   * interfaces：提供接口的字节码对象数组

   * h：提供代理对象要做的事情，通常写成InvocationHandler接口的匿名内部类

     ```java
     new InvocationHandler(){
         public Object invoke(Object proxy, Method method, Object[] args){
             return null;
         }
     }
     ```

     * invoke方法：我们每次调用代理对象的方法，本质就是调用这个invoke方法
     * 方法的参数：
       * proxy：最终生成的那个代理对象
       * method：我们调用的方法
       * args：我们调用方法时，传递的实参
     * 方法的返回值：
       * 我们调用代理对象的方法时，给我们返回的结果
