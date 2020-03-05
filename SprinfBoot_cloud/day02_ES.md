---
typora-copy-images-to: img
---

# day02_ES

## 学习目标

1. 能够完成创建索引的操作
2. 能够完成删除索引的操作
3. 能够完成创建映射的操作
4. 能够完成文档的增删改查
5. 能够完成文档的分页操作
6. 能够完成文档的高亮查询操作
7. 能够搭建Spring Data ElasticSearch的环境
8. 能够完成Spring Data ElasticSearch的基本增删改查操作
9. 能够掌握基本条件查询的方法命名规则 

## 第一章ElasticSearch常用编程操作 

### 1.索引相关操作 

参考上一次的项目工程 copy一份作为测试工程使用。

#### 1.1创建索引

```java
    @Test
    //创建索引
    public void fun01() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.创建索引
        client.admin().indices().prepareCreate("blog02").get();
        //3.释放资源
        client.close();
    }
```

#### 1.2删除索引

```java
    @Test
    //删除索引
    public void fun02() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.删除索引
        client.admin().indices().prepareDelete("blog02").get();
        //3.释放资源
        client.close();
    }
```

### 2.映射相关操作

#### 2.1映射格式

```json
"mappings" : {
    "article" : {
        "properties" : {
            "id" : { "type" : "long" },
            "title" : { "type" : "text" },
            "content" : { "type" : "text" }
        }
    }
}
```

#### 2.2创建映射

```java
    public void fun03() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.创建索引
        client.admin().indices().prepareCreate("blog02").get();

        //3.创建映射
        XContentBuilder builder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("article")
                .startObject("properties")
                .startObject("id").field("type", "long").field("store", "true").endObject()
                .startObject("title").field("type", "text").field("analyzer", "ik_smart").field("store", "true").endObject()
                .startObject("content").field("type", "text").field("analyzer", "ik_smart").field("store", "true").endObject()
                .endObject()
                .endObject()
                .endObject();
        PutMappingRequest mapping=Requests.putMappingRequest("blog02").type("article").source(builder);
        client.admin().indices().putMapping(mapping).get();

        //4.释放资源
        client.close();
    }
```



### 3.文档相关操作 

#### 3.1建立文档

##### 3.1.1通过XContentBuilder(复习)

```java
    @Test
    //创建文档(使用XContentBuilder)
    public void fun05() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.创建文档 {"id":1,"title":"",content:""}
        XContentBuilder builder = XContentFactory.jsonBuilder()
                .startObject()
                .field("id", 1)
                .field("title", "ElasticSearch是一个基于Lucene的搜索服务器")
                .field("content", "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。")
                .endObject();
        client.prepareIndex("blog02","article","1").setSource(builder).get();

        //3.释放资源
        client.close();
    }
```

##### 3.1.2使用Jackson转换实体

+ 创建Article实体

```java
public class Article implements Serializable {
    private long id;
    private String title;
    private String content;
    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
```

+ 在pom.xml添加jackson依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.8.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.8.1</version>
</dependency>
```

+ 代码实现

```java
    @Test
    //创建文档(使用Jackson转换实体)
    public void fun06() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.创建文档 {"id":1,"title":"",content:""}
        Article article = new Article(2, "搜索使用ES", "ES搜索很棒!");
        ObjectMapper objectMapper = new ObjectMapper();
        String jsonStr = objectMapper.writeValueAsString(article);
        client.prepareIndex("blog02","article",article.getId()+"").setSource(jsonStr).get();

        //3.释放资源
        client.close();
    }
```



#### 3.2修改文档 

##### 3.2.1使用prepareIndex修改文档 

```java
    @Test
    //修改文档
    public void fun07() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.创建文档 {"id":1,"title":"",content:""}
        Article article = new Article(2, "搜索使用ES", "ES搜索很棒!");
        ObjectMapper objectMapper = new ObjectMapper();
        String jsonStr = objectMapper.writeValueAsString(article);
        client.prepareIndex("blog02","article",article.getId()+"").setSource(jsonStr).get();

        //3.释放资源
        client.close();
    }
```

##### 3.2.2直接使用update修改文档 

```java
    @Test
    //修改文档
    public void fun08() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.创建文档 {"id":1,"title":"",content:""}
        Article article = new Article(3, "更新2:搜索使用ES", "更新2:ES搜索很棒!");
        ObjectMapper objectMapper = new ObjectMapper();
        String jsonStr = objectMapper.writeValueAsString(article);
        client.update(new UpdateRequest("blog02","article","2").doc(jsonStr)).get();
        //3.释放资源
        client.close();
    }
```

#### 3.3删除文档

##### 3.3.1通过prepareDelete 删除文档  

```java
    @Test
    //删除文档
    public void fun09() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.删除文档
        client.prepareDelete("blog02","article","2").get();

        //3.释放资源
        client.close();
    }
```

##### 3.3.2直接使用delete 删除文档 

```java
    @Test
    //删除文档
    public void fun10() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.删除文档
        client.delete(new DeleteRequest("blog02","article","2")).get();

        //3.释放资源
        client.close();
    }
```



#### 3.4查询文档 

##### 3.4.1批量插入数据(准备工作) 

```java
    @Test
    //批量创建文档(使用Jackson转换实体)
    public void fun11() throws Exception {
        //1.创建客户端对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));

        //2.创建文档
        ObjectMapper objectMapper = new ObjectMapper();
        for(int i =0;i<100;i++){
            Article article = new Article(i, i+"搜索工作SoEasy", i+"我们希望搜索解决方案要运行速度快，我们希望能有一个零配置和一个完全免费的搜索模式，我们希望能够简单地使用JSON通过HTTP来索引数据，我们希望我们的搜索服务器始终可用，我们希望能够从一台开始并扩展到数百台，我们要实时搜索，我们要简单的多租户，我们希望建立一个云的解决方案。因此我们利用Elasticsearch来解决所有这些问题及可能出现的更多其它问题。");
            String jsonStr = objectMapper.writeValueAsString(article);
            client.prepareIndex("blog02","article",article.getId()+"").setSource(jsonStr).get();
        }
        //3.释放资源
        client.close();
    }
```

##### 3.4.2文档的查询

+  matchAllQuery():查询全部


```java
    @Test
    //全部 matchAllQuery()
    public void fun12() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.matchAllQuery())
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }

        //4.释放资源
        client.close();
    }
```

+ queryStringQuery():字符串查询(分词查询)

```java
    @Test
    //字符串查询queryStringQuery()
    public void fun13() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.queryStringQuery("SoEasy 搜索").field("content").field("title",10).minimumShouldMatch("50%"))
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数=" + hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }

        //4.释放资源
        client.close();
    }
```

+ termQuery():词条查询

  Term Query为精确查询，在搜索时会整体匹配关键字，不再将关键字分词。 

```java
    @Test
    //词条查询termQuery()
    public void fun14() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.termQuery("title","搜索"))
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```

+ match Query :全文检索

  match Query即全文检索，它的搜索方式是先将搜索字符串分词，再使用各各词条从索引中搜索。
  match query与Term query区别是match query在搜索前先将搜索关键字分词，再拿各各词语去索引中搜索。 


+ wildcardQuery():模糊查询

```java
    @Test
    //模糊查询wildcardQuery()
    public void fun15() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.wildcardQuery("title","搜索*"))
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```

+ 相似度查询fuzzyQuery()

```java
   @Test
    //模糊查询fuzzyQuery()
    public void fun16() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.fuzzyQuery("title","soeaaa"))
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```

+ 范围查询rangeQuery()

```java
    @Test
    //范围查询rangeQuery()
    public void fun17() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.rangeQuery("id").gt(5).lt(10))
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```

##### 3.4.3布尔查询boolQuery（组合查询）【扩展】

###### 3.4.3.1相关的概念

+ 说明

```
must(QueryBuilders) : AND，求交集
mustNot(QueryBuilders): NOT
should(QueryBuilders):OR ，求并集
```

+ 在定义json：放置到Elasticsearch的插件中（DSL表达式）

```
{
	"query": {
		"bool": {
			"must": {
				"term": {
					"title": "elasticsearch"
				}
			},
			"must": {
				"range": {
					"id": {
						"from": 1,
						"to": 5
					}
				}
			}
		}
	}
}
```

 ![1552919080115](img/1552919080115.png)

###### 3.4.3.2代码实现

+ 查询5<id<20,并且titile中包含搜索的

```java
    @Test
    //布尔查询boolQuery()
    public void fun18() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.boolQuery().
                        must(QueryBuilders.rangeQuery("id").gt(5).lt(20)).
                        must(QueryBuilders.termQuery("title","搜索"))
                ).get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```





##### 3.4.5过虑器 

+ 过虑是针对搜索的结果进行过虑，过虑器主要判断的是文档是否匹配，不去计算和判断文档的匹配度得分，所以过虑器性能比查询要高，且方便缓存，推荐尽量使用过虑器去实现查询或者过虑器和查询共同使用。


```java
    @Test
    //过滤器
    public void fun21() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                //.setQuery(QueryBuilders.rangeQuery("id").gt(5).lt(10))
                .setQuery(QueryBuilders.boolQuery()
                        .filter(QueryBuilders.rangeQuery("id").gt(7).lt(30))
                        .filter(QueryBuilders.termQuery("title","搜索")))
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数=" + hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```

##### 3.4.6分页查询和排序 

+ ES支持分页查询，传入两个参数：from和size。

  form：表示起始文档的下标，从0开始。
  size：查询的文档数量。 

+ 可以在字段上添加一个或多个排序，支持在keyword、date、float等类型上添加，text类型的字段上不允许添加排
  序。 

```java
    @Test
    //分页 排序
    public void fun19() throws Exception {
        //1.创建客户端连接对象,设置连接地址
        PreBuiltTransportClient client = new PreBuiltTransportClient(Settings.EMPTY);
        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));
        //2.设置查询条件
        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.matchAllQuery()).setFrom(0).setSize(5).addSort("id", SortOrder.ASC)
                .get();
        SearchHits hits = searchResponse.getHits();
        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数="+hits.totalHits);
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
          c
            System.out.println("------------------");
        }
        //4.释放资源
        client.close();
    }
```

#### 3.5查询结果高亮操作 

##### 3.5.1 什么是高亮显示

在进行关键字搜索时，搜索出的内容中的关键字会显示不同的颜色，称之为高亮

+ 百度搜索关键字"传智播客"

![1552828421636](img/1552828421636.png)

+ 京东商城搜索"笔记本"

  ![1552828436266](img/1552828436266.png)

+ 在百度搜索"elasticsearch",查看页面源码分析

  ![1552828452324](img/1552828452324.png)

  ![1552828465385](img/1552828465385.png)

  


##### 3.5.2高亮显示的html分析

通过开发者工具查看高亮数据的html代码实现：

ElasticSearch可以对查询出的内容中关键字部分进行标签和样式的设置，但是你需要告诉ElasticSearch使用什么标签对高亮关键字进行包裹呢？

使用`<em>高亮内容</em>` 

##### 3.5.3 高亮显示代码实现

```java

    @Test
    //高亮
    public void fun20() throws Exception {
        // 创建Client
        Settings settings = Settings.EMPTY;
        Client client = new PreBuiltTransportClient(settings)
                .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"),9300));

        // 组织高亮的设置条件
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<em>");// 设置前缀
        highlightBuilder.postTags("</em>");// 设置后缀
        highlightBuilder.field("title");// 设置在哪个字段上进行高亮

        SearchResponse searchResponse = client.prepareSearch("blog02").setTypes("article")
                .setQuery(QueryBuilders.termQuery("title","搜索"))
                .setFrom(0).setSize(5) //设置分页
                .highlighter(highlightBuilder) //设置高亮
                .addSort("id", SortOrder.ASC) //设置排序
                .get();
        SearchHits hits = searchResponse.getHits();

        //3.处理结果
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println("条数=" + hits.totalHits);
        for (SearchHit hit : hits) {
            Article article = objectMapper.readValue(hit.getSourceAsString(), Article.class);
            System.out.println(article);

            //取出高亮
            HighlightField highlightField = hit.getHighlightFields().get("title");
            Text[] text = highlightField.getFragments();
            String title = "";
            for (Text txt : text) {
                title+=txt;
            }
            article.setTitle(title);
            System.out.println(article);
        }
        //4.释放资源
        client.close();
    }
```





## 第二章Spring Data ElasticSearch 

### 1.Spring Data ElasticSearch简介

#### 1.1什么是Spring Data

​	Spring Data是一个用于简化数据库访问，并支持云服务的开源框架。其主要目标是使得对数据的访问变得方便快捷，并支持map-reduce框架和云计算数据服务。 Spring Data可以极大的简化JPA的写法，可以在几乎不用写实现的情况下，实现对数据的访问和操作。除了CRUD外，还包括如分页、排序等一些常用的功能。

​	Spring Data的官网：[http://projects.spring.io/spring-data/](http://projects.spring.io/spring-data/)

​	Spring Data常用的功能模块如下：

![1552828591302](img/1552828591302.png)

![1552828597851](img/1552828597851.png)

#### 1.2什么是Spring Data ElasticSearch

​	Spring Data ElasticSearch 基于 spring data API 简化 elasticSearch操作，将原始操作elasticSearch的客户端API 进
行封装 。Spring Data为Elasticsearch项目提供集成搜索引擎。Spring Data Elasticsearch POJO的关键功能区域为
中心的模型与Elastichsearch交互文档和轻松地编写一个存储库数据访问层。
​	官方网站：http://projects.spring.io/spring-data-elasticsearch/ 

### 2.Spring Data ElasticSearch入门

#### 2.1需求

需求: 保存Article

步骤:

1. 创建Maven工程(jar),在pom文件导入坐标
2. 创建pojo, 添加注解
3. 创建Dao接口继承ElasticsearchRepository
4. 创建配置文件进行配置
5. 测试

#### 2.2代码实现

+ 创建Maven工程(jar),导入坐标

```xml
    <dependencies>
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
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-elasticsearch</artifactId>
            <version>3.0.7.RELEASE</version>
        </dependency>
    </dependencies>
```

+ 创建pojo, 添加注解

```java
/**
 * @Document：放置到类上
 *    indexName = "blog1"：表示索引的名称，（小写）
 *    type = "article"：表示类型
 * @Id：放置到字段id上
 *    表示该字段的值存放到索引库的_id字段上，表示主键
 * @Field：放置到字段上
 *    store = true：表示该字段的值存储到索引库
 *    index = true：表示该字段的值要建立索引用于搜索
 *    analyzer = "ik_smart"：建立索引的时候使用什么分词器
 *    searchAnalyzer = "ik_smart"：数据搜索的时候使用什么分词器（可以不写）
 *    type = FieldType.Text：存放字段的数据类型
 */
@Document(indexName = "blog03",type = "article")
public class Article implements Serializable {

    @Id
    private Long id;

    @Field(index = true,searchAnalyzer = "ik_smart",analyzer = "ik_smart",store = true,type = FieldType.Text)
    private String title;

    @Field(index = true,searchAnalyzer = "ik_smart",analyzer = "ik_smart",store = true,type = FieldType.Text)
    private String content;

    public Article() {
    }
    public Article(long id, String title, String content) {
        this.id = id;
        this.title = title;
        this.content = content;
    }


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Article{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", content='" + content + '\'' +
                '}';
    }
}
```

+ 创建Dao接口继承ElasticsearchRepository

```java
public interface ArticleDao  extends ElasticsearchRepository<Article,Long>{

}
```

+ 创建配置文件进行配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/data/elasticsearch http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!-- 1：扫描Dao包，自动创建实例，扫描所有继承ElasticsearchRepository接口的接口 -->
    <elasticsearch:repositories base-package="com.itheima.dao"></elasticsearch:repositories>
    <!--2.配置elasticSearch的连接-->
    <elasticsearch:transport-client id="client" cluster-nodes="127.0.0.1:9300" cluster-name="elasticsearch"></elasticsearch:transport-client>
    <!--3.注册模版-->
    <bean id="elasticsearchTemplate" class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="client"></constructor-arg>
    </bean>
</beans>
```

+ 测试

```java
@RunWith(value = SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class EsTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @Autowired
    private ArticleDao articleDao;

    @Test
    //创建索引和映射
    public void fun01(){
        elasticsearchTemplate.createIndex(Article.class);
        elasticsearchTemplate.putMapping(Article.class);
    }

    @Test
    //保存文章
    public void fun02(){
        Article article = new Article(1, "测试title", "测试内容");
        articleDao.save(article);
    }

}
```

### 3.Spring Data ElasticSearch常见操作

#### 3.1CRUD

+ 新增,

```
@Test
//保存文章
public void fun02(){
  Article article = new Article(1, "测试title", "测试内容");
  articleDao.save(article);
}
```

+ 删除

```
@Test
//删除
public void fun03(){
	articleDao.deleteById(4L);
}
```

+  更新;  没有id对应的数据,就是新增; 有当前id对应的数据,就是更新

```java
    @Test
    //修改
    public void fun04(){
        Article article = new Article(4, "测试标题4", "测试内容4");
        articleDao.save(article);
    }
```

+ 根据id查询

```java
    @Test
    //根据id查询
    public void fun05(){
        Article article = articleDao.findById(1L).get();
        System.out.println(article);
    }
```

+ 查询所有

```java

    @Test
    //根据所有
    public void fun06(){
        Iterable<Article> iterable = articleDao.findAll();
        for (Article article : iterable) {
            System.out.println(article);
        }
    }
```

+ 排序查询

```java
    @Test
    //排序
    public void fun07(){
        Iterable<Article> iterable = articleDao.findAll( Sort.by(Sort.Order.asc("id")));
        for (Article article : iterable) {
            System.out.println(article);
        }
    }
```

+ 分页查询

```java
    @Test
    //分页查询
    public void fun08(){
        Pageable pageable = PageRequest.of(0,2);
        Page<Article> page = articleDao.findAll(pageable);
        System.out.println("总数量="+page.getTotalPages());
        System.out.println("结果="+page.getContent());
    }
```

#### 3.2自定义查询

+ 常用查询命名规则

| **关键字**       | **命名规则**              | **解释**              | **示例**                |
| ------------- | --------------------- | ------------------- | --------------------- |
| and           | findByField1AndField2 | 根据Field1和Field2获得数据 | findByTitleAndContent |
| or            | findByField1OrField2  | 根据Field1或Field2获得数据 | findByTitleOrContent  |
| is            | findByField           | 根据Field获得数据         | findByTitle           |
| not           | findByFieldNot        | 根据Field获得补集数据       | findByTitleNot        |
| between       | findByFieldBetween    | 获得指定范围的数据           | findByPriceBetween    |
| lessThanEqual | findByFieldLessThan   | 获得小于等于指定值的数据        | findByPriceLessThan   |

+ 测试

```java
public interface ArticleDao  extends ElasticsearchRepository<Article,Long>{

    //根据title模糊查询
    List<Article> findByTitleLike(String title);

    //根据title模糊查询,根据id降序
    List<Article> findByTitleLikeOrderByIdAsc(String title);

    //模块查询,排序加分页
    Page<Article> findByTitleLikeOrderByIdAsc(String title,Pageable pageable);

}
```