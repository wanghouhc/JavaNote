# day01

## 课程目标

1. 能够理解ElasticSearch的作用（第一章）

2. 能够安装ElasticSearch服务（第二章）

3. 能够理解ElasticSearch的相关概念（第三章）

4. 能够使用java客户端操作ElasticSearch（第四章）

5. 能够理解分词器的作用（第五章）

6. 能够使用ElasticSearch集成IK分词器（第五章）

7. 能够使用restful技术操作操作ElasticSearch（附加）

## 1 ElasticSearch简介

### 1.1 什么是ElasticSearch？

​	ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于[云计算](https://baike.baidu.com/item/%E4%BA%91%E8%AE%A1%E7%AE%97/9969353)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

​	我们建立一个网站或应用程序，并要添加搜索功能，但是想要完成搜索工作的创建是非常困难的。我们希望搜索解决方案要运行速度快，我们希望能有一个零配置和一个完全免费的搜索模式，我们希望能够简单地使用JSON通过HTTP来索引数据，我们希望我们的搜索服务器始终可用，我们希望能够从一台开始并扩展到数百台，我们要实时搜索，我们要简单的多租户，我们希望建立一个云的解决方案。因此我们利用Elasticsearch来解决所有这些问题及可能出现的更多其它问题。

​	ElaticSearch，简称为es， es是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。es也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

### 1.2 ElasticSearch的使用案例

2013年初，GitHub抛弃了Solr，采取ElasticSearch 来做PB级的搜索。 “GitHub使用ElasticSearch搜索20TB的数据，包括13亿文件和1300亿行代码”

维基百科：启动以elasticsearch为基础的核心搜索架构

SoundCloud：“SoundCloud使用ElasticSearch为1.8亿用户提供即时而精准的音乐搜索服务”

百度：百度目前广泛使用ElasticSearch作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部20多个业务线（包括casio、云析分、网盟、预测、文库、直达号、钱包、风控等），单集群最大100台机器，200个ES节点，每天导入30TB+数据

新浪：使用ES 分析处理32亿条实时日志

阿里：使用ES 构建挖财自己的日志采集和分析体系



### 1.3 ElasticSearch对比Solr

Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;

Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式；

Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能多由第三方插件提供；

Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch；

Elasticsearch支持RestFul风格编程（uri的地址，就可以检索），Solr暂不支持；



## 2 ElasticSearch安装与启动

### 2.1下载ES压缩包

​	ElasticSearch分为Linux和Window版本，基于我们主要学习的是ElasticSearch的Java客户端的使用，所以我们课程中使用的是安装较为简便的Window版本，项目上线后，公司的运维人员会安装Linux版的ES供我们连接使用。

ElasticSearch的官方地址： <https://www.elastic.co/products/elasticsearch>



![1555749598297](images/1555749598297.png)



![1555749611853](images/1555749611853.png)



![1555749621204](images/1555749621204.png)



![1555749633290](images/1555749633290.png)



已经下载好了如下图：

![1555749674356](images/1555749674356.png)

### 2.2 安装ES服务

​	Window版的ElasticSearch的安装很简单，类似Window版的Tomcat，解压开即安装完毕，解压后的ElasticSearch的目录结构如下：

![1555749728053](images/1555749728053.png)

注意：ES的目录不要出现中文，也不要有特殊字符。



### 2.3 启动ES服务

![1555749776243](images/1555749776243.png)



启动：

![1555749789317](images/1555749789317.png)

注意：

9300是tcp通讯端口，集群间和TCPClient都执行该端口，可供java程序调用；

9200是http协议的RESTful接口 。

通过浏览器访问ElasticSearch服务器，看到如下返回的json信息，代表服务启动成功：

![1555749812443](images/1555749812443.png)

注意事项：

（1）ElasticSearch5是使用java开发的，且本版本的es需要的jdk版本要是1.8以上，所以安装ElasticSearch之前保证JDK1.8+安装完毕，并正确的配置好JDK环境变量，否则启动ElasticSearch失败



（2）出现闪退，通过路径访问发现“空间不足”

修改：解压目录下的/config/jvm.options文件



![1555749886519](images/1555749886519.png)

### 2.4 安装ES的图形化界面插件

​	ElasticSearch不同于Solr自带图形化界面，我们可以通过安装ElasticSearch的head插件，完成图形化界面的效果，完成索引数据的查看。

**elasticsearch-5-*以上版本安装head插件，需要先安装 nodejs 以及 grunt** 

#### 2.4.1 安装nodejs

![1555857158461](images/1555857158461.png)

下载nodejs：<https://nodejs.org/en/download/> 

![1555750063046](images/1555750063046.png)

双击如图所示msi文件，下一步即可。安装成功后，执行cmd命令查看版本：

```
node -v
```

![1555750096678](images/1555750096678.png)



安装 grunt:

​	Grunt是基于Node.js的项目构建工具；是基于javaScript上的一个很强大的前端自动化工具，基于NodeJs用于自动化构建、测试、生成文档的项目管理工具,在cmd命令行中执行以下命令：

![1555857306682](images/1555857306682.png)



```shell
npm install -g grunt-cli
```



#### 2.4.2 安装ES head 插件

1）下载head插件：<https://github.com/mobz/elasticsearch-head> 

![1555750240077](images/1555750240077.png)

![1555750276375](images/1555750276375.png)



2）解压zip

解压zip包,并在解压目录下执行cmd命令：

![1555750444224](images/1555750444224.png)

```
npm install
```



执行完成 等待结果：



![1555750489857](images/1555750489857.png)

ps:如果安装不成功或者安装速度慢，可以使用淘宝的镜像进行安装：

```shell
npm install -g cnpm –-registry=https://registry.npm.taobao.org
```

后续使用的时候，只需要把npm  xxx   换成  cnpm xxx 即可



#### 2.4.3 配置ES支持head插件

修改elasticsearch/config下的配置文件：elasticsearch.yml，增加以下配置：

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
network.host: 127.0.0.1
```

注意有空格:

![1555750618227](images/1555750618227.png)

解释：

```
http.cors.enabled: true：此步为允许elasticsearch跨域访问，默认是false。
http.cors.allow-origin: "*"：表示跨域访问允许的域名地址（*表示任意）。
network.host:127.0.0.1：主机域名
```



#### 2.4.4 访问head插件

1） 重启ES服务器

![1555752496047](images/1555752496047.png)

2）进入head插件的目录 执行cmd命令：

```
npm run start
```

![1555752600757](images/1555752600757.png)

## 3 ElasticSearch相关概念

### 3.1 概述

​	Elasticsearch是面向文档(document oriented)的，这意味着它可以存储整个对象或文档(document)。然而它不仅仅是存储（store），还会索引(index)每个文档的内容使之可以被搜索。在Elasticsearch中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。Elasticsearch比较传统关系型数据库如下：

```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields
```



![1555851901208](images/1555851901208.png)



### 3.2 核心概念

#### 3.2.1 索引 index

​	一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（必须全部是小写字母的），并且当我们要对对应于这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。

#### 3.2.2. 类型 type

​	在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。比如说，我们假设你运营一个博客平台并且将你所有的数据存储到一个索引中。在这个索引中，你可以为用户数据定义一个类型，为博客数据定义另一个类型，当然，也可以为评论数据定义另一个类型。

#### 3.2.3. 文档 document

​	一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式。

在一个index/type里面，你可以存储任意多的文档。注意，尽管一个文档，物理上存在于一个索引之中，文档必须被索引/赋予一个索引的type。

#### 3.2.4. 字段Field

​	相当于是数据表的字段，对文档数据根据不同属性进行的分类标识

#### 3.2.5. 映射 mapping

​	mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分词器、是否被索引等等，这些都是映射里面可以设置的，其它就是处理es里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。

例如：

![1555852157614](images/1555852157614.png)

#### 3.2.6. 接近实时 NRT

​	Elasticsearch是一个接近实时的搜索平台。这意味着，从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟（通常是1秒以内）

#### 3.2.7. 集群 cluster

​	一个集群就是由一个或多个节点组织在一起，它们共同持有整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

#### 3.2.8. 节点 node

​	一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能。和集群类似，一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于Elasticsearch集群中的哪些节点。

一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。

在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何Elasticsearch节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。

#### 3.2.9. 分片 和 复制 shards & replicas

​	一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片很重要，主要有两方面的原因： 1）允许你水平分割/扩展你的内容容量。 2）允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量。

至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的。

在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。

（后面大家自己看即可）

复制之所以重要，有两个主要原因： 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行。总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。

默认情况下，Elasticsearch中的每个索引被分片5个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有5个主分片和另外5个复制分片（1个完全拷贝），这样的话每个索引总共就有10个分片。

![1555852281799](images/1555852281799.png)





## 4. Java操作ElasticSearch

### 4.1 搭建ElasticSearch操作环境 

#### 4.1.1 创建Maven工程

![1556180254417](images/1556180254417.png)

#### 4.1.2 引入ElasticSearch坐标

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>5.6.8</version>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>transport</artifactId>
        <version>5.6.8</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-to-slf4j</artifactId>
        <version>2.9.1</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.24</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.12</version>
    </dependency>

</dependencies>
```

### 4.2 操作ElasticSearch

#### 4.2.1 新建索引+添加文档

使用创建索引+自动创建映射（Elasticsearch帮助我们自动建立映射，后续讲完分词器后，手动建立映射）

```java
public class EsTest {

    @Test
    //创建索引+插入文档
    public void fun01() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        TransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));

        //2.创建文档数据  {id:1,title:"xxx",content:"xxxxxx"}
        /*XContentBuilder builder = XContentFactory.jsonBuilder().startObject()
                .field("id", 1)
                .field("title", "ElasticSearch是一个基于Lucene的搜索服务器")
                .field("content", "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。")
                .endObject();*/
        Map<String,Object> map = new HashMap<String, Object>();
        map.put("id",1);
        map.put("title","ElasticSearch是一个基于Lucene的搜索服务器");
        map.put("content","ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。");

        //3.创建索引、创建文档类型、设置唯一主键,设置文档
        client.prepareIndex("blog","article","1").setSource(map).get();

        //4.释放资源
        client.close();

    }


}
```



head插件查看信息：http://localhost:9100



#### 4.2.2 查询全部 matchAllQuery()

```java
@Test
//查询所有的文档
public void fun02() throws Exception {
    //1.创建客户端连接对象,设置连接地址
    PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
    client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
    //2.指定索引库和类型
    SearchRequestBuilder searchRequestBuilder = client.prepareSearch("blog").setTypes("article");

    //3.设置查询条件,进行查询
    SearchResponse searchResponse = searchRequestBuilder.setQuery(QueryBuilders.matchAllQuery()).get();

    //4.处理结果
    SearchHits hits = searchResponse.getHits();
    System.out.println("条数="+hits.totalHits);
    for (SearchHit hit : hits) {
        System.out.println("id="+hit.getId());
        System.out.println("标题="+hit.getSource().get("title"));
        System.out.println("内容="+hit.getSource().get("content"));
        System.out.println("---------------");
    }

    //5.关闭资源
    client.close();
}
```
特点: 

​	自动的分词 再搜索

#### 4.2.3 字符串搜索StringQurey()

```java
@Test
//字符串查询
public void test3() throws Exception {
    //创建客户端访问对象
    TransportClient client = new PreBuiltTransportClient(Settings.EMPTY).addTransportAddress(
        new InetSocketTransportAddress(
            InetAddress.getByName("127.0.0.1"), 9300));
    //设置查询条件
    SearchResponse searchResponse = client.prepareSearch("blog").setTypes("article")
        .setQuery(QueryBuilders.queryStringQuery("搜索").field("title")) // 默认在所有的字段上进行搜索，搜索“搜素”；如果添加.field("title")：表示只在title字段进行搜索
        .get();
    //处理结果
    SearchHits hits = searchResponse.getHits(); //获得命中目标，即查询到了多少个对象
    System.out.println("共查询"+hits.getTotalHits()+"条");
    Iterator<SearchHit> ite = hits.iterator();
    while(ite.hasNext()){
        SearchHit searchHit = ite.next();
        System.out.println(searchHit.getSourceAsString());
        System.out.println(searchHit.getSource().get("title"));
    }
    //关闭资源
    client.close();
}

```

查看结果能搜索出来结果。



#### 4.2.4 词条查询termQuery()

什么是词条:

![1556180612562](images/1556180612562.png)

![1556180633249](images/1556180633249.png)



代码

```java
@Test
    //词条查询
    public void fun04() throws Exception {
        //1.创建客户端连接对象, 设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));

        //2.指定索引库和内容
        SearchRequestBuilder searchRequestBuilder = client.prepareSearch("blog").setTypes("article");

        //3.设置查询添加进行查询
        SearchResponse searchResponse = searchRequestBuilder.setQuery(QueryBuilders.termQuery("title","搜索")).get();

        //4.处理结果
        SearchHits hits = searchResponse.getHits();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println("id="+hit.getId());
            System.out.println("title="+hit.getSource().get("title"));
            System.out.println("content="+hit.getSource().get("content"));
        }

        //5.释放资源
        client.close();
    }
```

#### 4.2.5 模糊查询wildcardQuery()

```java
 @Test
    /**
     *  模糊查询
     *  *匹配任意字符
     *  ?匹配1个字符
     */
    public void fun05() throws Exception {
        //1.创建客户端连接对象, 设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));

        //2.指定索引库和内容
        SearchRequestBuilder searchRequestBuilder = client.prepareSearch("blog").setTypes("article");

        //3.设置查询添加进行查询
        SearchResponse searchResponse = searchRequestBuilder.setQuery(QueryBuilders.wildcardQuery("title","*搜索*")).get();

        //4.处理结果
        SearchHits hits = searchResponse.getHits();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println("id="+hit.getId());
            System.out.println("title="+hit.getSource().get("title"));
            System.out.println("content="+hit.getSource().get("content"));
        }

        //5.释放资源
        client.close();
    }
```



#### 4.2.6 小结

字符串查询  默认查询所有字段上的内容。字符串查询“搜索”时能搜索出结果。因为先进行分词成 一个个单词再进行查询，能查询到。

词条查询 查询指定字段上的内容。词条查询“搜索”时查询不结果。因为查询时 不分词 ，索引中没有“搜索”的词条，所以查询不到。

模糊查询（通配符查询） 

​	使用 * 表示任意字符 可以占一个字符也可以不占。

​	使用 ? 表示任意字符 但是一定要占一个字符。



## 5 IK 分词器和ElasticSearch集成使用

### 5.1 上述查询存在问题分析

在进行词条查询时，我们搜索"搜索"却没有搜索到数据！

原因：默认是单字分词，默认的中文分词是将每个字看成一个词，这显然是不符合要求的，所以我们需要安装中文分词器来解决这个问题。解决方案：使用IK分词器

### 5.2 IK分词器

#### 5.2.1 什么是IK分词器

IK分词是一款国人开发的相对简单的中文分词器。虽然开发者自2012年之后就不在维护了，但在工程应用中IK算是比较流行的一款！

#### 5.2.2 IK分词器安装

下载地址：

```
https://github.com/medcl/elasticsearch-analysis-ik/releases
```

 我们已经提供：

![1556181020394](images/1556181020394.png)



- 先将其解压，将解压后的elasticsearch文件夹重命名文件夹为ik
- 将ik文件夹拷贝到elasticsearch/plugins 目录下。
- 重新启动，即可加载IK分词器 

#### 5.2.3 IK分词器测试

IK提供了两个分词算法ik_smart 和 ik_max_word. 其中 ik_smart 为最少切分(智能切分)，ik_max_word为最细粒度划分 

+ 第一种：最小切分
+ 第二种：种最细切分

测试第一种：浏览器中输入：

```
http://127.0.0.1:9200/_analyze?analyzer=ik_smart&pretty=true&text=我是程序员
```

![1556181174707](images/1556181174707.png)

测试第二种：浏览器输入：

```
http://127.0.0.1:9200/_analyze?analyzer=ik_max_word&pretty=true&text=我是程序员
```

![1556181210421](images/1556181210421.png)

#### 5.2.4 自定义词库

​	我们现在测试"传智播客"，浏览器的测试效果如下  http://127.0.0.1:9200/_analyze?analyzer=ik_smart&pretty=true&text=传智播客 

```
{
  "tokens" : [
    {
      "token" : "传",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "智",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "播",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "客",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 3
    }
  ]
}
```

默认的分词并没有识别“传智播客”是一个词。如果我们想让系统识别“传智播客”是一个词，需要编辑自定义词库 .



- 进入elasticsearch/plugins/ik/config目录 
- 新建一个my.dic文件，编辑内容(以utf8无bom保存, 如果不行加一些换行)

```
传智播客
```

+ 修改IKAnalyzer.cfg.xml（在ik/config目录下） 

```
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!‐‐用户可以在这里配置自己的扩展字典 ‐‐>
    <entry key="ext_dict">my.dic</entry>
    <!‐‐用户可以在这里配置自己的扩展停止词字典‐‐>
    <entry key="ext_stopwords"></entry>
</properties>
```

+ 重新启动elasticsearch,通过浏览器测试分词效果 

## 疑问？

​	在ElasticSearch没有索引的情况下，插入文档，默认创建索引和索引映射是无法使用IK分词器的。

要想让索引库可以使用ik分词器，必须要手动创建映射的配置。如何手动创建映射呢？下回分解。