---
typora-copy-images-to: images
---

# lucene-全文检索技术

## 1 课程计划

+ lucene的介绍
  + 什么是lucene
  + lucene的应用场景
  + 全文检索的定义
+ lucene实现全文检索的流程
+ 入门案例
  + 环境准备
  + 索引流程
  + 搜索流程
+ Field域介绍
+ 索引维护以及lucene的语法
+ 中文分词器使用

## 2 lucene的介绍

### 2.1 什么是lucene

​	Lucene是apache下的一个开源的全文检索引擎工具包(jar包)。它的目的是为软件开发人员提供一个简单易用的工具包，以方便在目标系统中实现全文检索的功能。

### 2.2 lucene的应用场景

+ 搜索引擎 例如:百度 google.
+ 站内搜索 各大电商系统（jd taobao）
+ 文件系统搜索（windows中文件系统搜索）



## 3 全文检索的介绍

### 3.1 全文检索的定义

​	全文检索首先将要查询的目标文档中的词提取出来，组成索引，通过查询索引达到搜索目标文档的目的。这种先建立索引，再对索引进行搜索的过程就叫**全文检索（Full-text Search）**



### 3.2 lucene实现全文检索的流程

![1541427467858](images/1541427467858.png)

全文检索分成两部分：1.索引流程 2 搜索的流程

1.索引流程：采集数据-->构建文档-->分析文档-->创建索引

2.搜索流程：用户搜索界面-->创建查询对象-->执行搜索-->渲染结果



### 3.3 采集数据

#### 3.3.1 为什么要采集数据

全文检索要搜索的数据信息格式是多种多样的，比如：搜索引擎（百度， google），通过搜索引擎网站能搜索互联网上的网页(html)、互联网上的音乐(mp3..)、视频(avi..mp4)、pdf电子书等。这些数据称为非结构化数据。

什么是非结构化数据？

	结构化数据：指具有固定格式或有限长度的数据，如数据库，元数据等。
	非结构化数据：指不定长或无固定格式的数据，如邮件，word文档等。


如何对非结构化数据搜索？

	由于结构化数据是固定格式，所以就可以针对固定格式的数据设计算法来搜索，比如数据库like查询，like查询采用顺序扫描法，使用关键字匹配内容，对于内容量大的like查询速度慢。
	需要将所有要搜索的非结构化数据通过技术手段采集到一个固定的地方，将这些非结构化的数据想办法组成结构化的数据，再以一定的算法去搜索。


总结一句话：由于数据是格式各样的，而需要使用lucene，就需要按照lucene的语法将数据封装到文档对象，这样就可以按照lucene的语法来实现搜索了。



#### 3.3.2 如何采集数据

+ 网页采集； 通过爬虫项目爬取网站的网页数据然后解析。爬虫项目：Nutch,heritrix 等爬虫项目。百度蜘蛛也是其中之一。
+ 数据库采集；通过数据库驱动程序例如JDBC来实现从数据库中查询数据。



## 4 入门案例

### 4.1 准备工作

需求：

​	使用lucene来实现**图书类商品的索引和搜索**。

分析：

​	我们采用数据库采集的方式来采集数据。

环境准备

​	jdk  1.8  

​	数据库 mysql

​	lucene 4.10.3

数据库准备

![1541429570839](images/1541429570839.png)

直接使用mysql的客户端导入如图所示的sql脚本即可。

### 4.2 图书表索引文件的逻辑结构说明

​	首先要搞清楚，在lucene中数据是存储在哪的。这里以Book表为例子来说明：

![1541430899336](images/1541430899336.png)



+ 多个document组成所有的数据
+ 一个document由多个field来组成.

Field:具体是由列名 和列名所对应的值组成。
不同document之间可以存在不通的数量的Field.建议不同的documen有一样field数量。



------

文档域：

对非结构化的数据统一格式为document文档格式，一个文档有多个field域，不同的文档其field的个数可以不同，建议相同类型的文档包括相同的field。

本例子一个document对应一条 book表的记录。

------

索引域：

用于搜索，搜索程序将从索引域中搜索一个一个词，根据词找到对应的文档。

将Document中的Field的内容进行分词，将分好的词创建索引。

索引域包括两个部分：

         一部分是：Field域的name

         一部分是：要搜索的内容

比如：bookname:java   要从域的名称为bookname上搜索内容为java的文档对象。



### 4.3 lucene的下载

Lucene是开发全文检索功能的工具包，使用时从官方网站下载，并解压。

 

官方网站：[http://lucene.apache.org/](http://lucene.apache.org/)

 

下载地址：[http://archive.apache.org/dist/lucene/java/](http://archive.apache.org/dist/lucene/java/)

 

下载版本：4.10.3

JDK要求：1.7以上（从版本4.8开始，不支持1.7以下）

参考资料中：

```
E:\courses\lucene\资料\lucene-4.10.3.zip
```



解压关注的目录：

![1541431392263](images/1541431392263.png)





### 4.4 工程搭建

#### 4.4.1 创建Maven工程





![1541431459976](images/1541431459976.png)

#### 4.4.2 pom.xml中添加依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>itheima_luccene_day1</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>itheima_luccene_day1</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-core</artifactId>
            <version>4.10.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-analyzers-common</artifactId>
            <version>4.10.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-queryparser</artifactId>
            <version>4.10.3</version>
        </dependency>

   </dependencies>
</project>

```

### 4.5 索引流程

流程分析：采集数据  创建文档  分析文档 创建索引



#### 4.5.1 采集数据

创建POJO

创建DAO 

创建DAO的实现类



```java
public class Book {
	// 图书ID
	private Integer id;
	// 图书名称
	private String name;
	// 图书价格
	private Float price;
	// 图书图片
	private String pic;
	// 图书描述
	private String description;
	//getter setter
}

```



```java
public interface BookDao {
	// 图书查询
	public List<Book> queryBookList();
}

```



```java
public class BookDaoImpl implements BookDao {

	@Override
	public List<Book> queryBookList() {
		// 数据库链接
		Connection connection = null;

		// 预编译statement
		PreparedStatement preparedStatement = null;

		// 结果集
		ResultSet resultSet = null;

		// 图书列表
		List<Book> list = new ArrayList<Book>();

		try {
			// 加载数据库驱动
			Class.forName("com.mysql.jdbc.Driver");
			// 连接数据库
			connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/book", "root", "itcast");

			// SQL语句
			String sql = "SELECT * FROM book";
			// 创建preparedStatement
			preparedStatement = connection.prepareStatement(sql);

			// 获取结果集
			resultSet = preparedStatement.executeQuery();

			// 结果集解析
			while (resultSet.next()) {
				Book book = new Book();
				book.setId(resultSet.getInt("id"));
				book.setName(resultSet.getString("name"));
				book.setPrice(resultSet.getFloat("price"));
				book.setPic(resultSet.getString("pic"));
				book.setDescription(resultSet.getString("description"));
				list.add(book);
			}

		} catch (Exception e) {
			e.printStackTrace();
		}finally {
			if (resultSet!=null) {
				try {
					resultSet.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (preparedStatement!=null) {
				try {
					preparedStatement.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if(connection!=null){
				try {
					connection.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
		return list;
	}
}

```



#### 4.5.2 创建索引

创建文档，分析文档，创建索引的流程，如下代码实现。

```java
public void createIndex() throws IOException {
		// 第一步采集数据：(jdbc采集数据)
		BookDao dao = new BookDaoImpl();
		List<Book> queryBookList = dao.queryBookList();
		// 将数据采集放到docment对象中
		Document doc = null;
		List<Document> docList = new ArrayList<>();
		for (Book book : queryBookList) {
			// 根据需求创建不同的field
			doc = new Document();
			Field id = new TextField("id", book.getId().toString(), Store.YES);
			Field name = new TextField("name", book.getName(), Store.YES);
			Field price = new TextField("price", book.getPrice().toString(), Store.YES);
			Field pic = new TextField("pic", book.getPic(), Store.YES);
			Field desc = new TextField("description", book.getDescription(), Store.YES);
			// 将field域，添加到文档对象中
			doc.add(id);
			doc.add(name);
			doc.add(price);
			doc.add(pic);
			doc.add(desc);
			docList.add(doc);
		}
		//创建一个indexWriter对象(通过它反向推理出需要的条件)
		//先构造一个directory  指定索引库的位置，一般使用文件系统的目录。
		FSDirectory directory = FSDirectory.open(new File("G:\\index01"));
		//创建一个分词器 对文档中的Field域进行分词
		Analyzer analyzer = new StandardAnalyzer();//标准分词器
		IndexWriterConfig config = new IndexWriterConfig(Version.LATEST, analyzer);
		IndexWriter writer = new IndexWriter(directory,config);
		//创建索引
		for (Document document : docList) {
			writer.addDocument(document);
		}
		//关闭资源
		writer.close();
	}

```



#### 4.5.3 创建文档和索引的过程

![1541431836258](images/1541431836258.png)



IndexWriter是索引过程的核心组件，通过IndexWriter可以创建新索引、更新索引、删除索引操作。IndexWriter需要通过Directory对索引进行存储操作。

Directory描述了索引的存储位置，底层封装了I/O操作，负责对索引进行存储。它是一个抽象类，它的子类常用的包括FSDirectory（在文件系统存储索引）、RAMDirectory（在内存存储索引）。





### 4.6 分词器

在对Document中的内容索引之前需要使用分词器（analyzer）进行分词，主要过程就是分词、过虑两步。

+ 分词:就是将采集到的文档内容切分成一个一个的词，具体应该说是将Document中Field的value值切分成一个一个的词
+ 过滤:包括去除标点符号、去除停用词（的、是、a、an、the等）、大写转小写、词的形还原（复数形式转成单数形式、过去式转成现在式。。。）等。

```
什么是停用词？停用词是为节省存储空间和提高搜索效率，搜索引擎在索引页面或处理搜索请求时会自动忽略某些字或词，这些字或词即被称为Stop Words(停用词)。比如语气助词、副词、介词、连接词等，通常自身并无明确的意义，只有将其放入一个完整的句子中才有一定作用，如常见的“的”、“在”、“是”、“啊”等。
```

注意：由于语言不同分词器的切分规则也不同，本例子使用StandardAnalyzer，它可以对用英文进行分词。

例如源文档内容为：

```
Lucene is a Java full-text search engine. 
```

分词之后：

```java
lucene 
java 
full
text 
search 
engine

```

以下是org.apache.lucene.analysis.standard.standardAnalyzer的部分源码

```java
@Override
protected TokenStreamComponents createComponents(final String fieldName, final Reader reader) {
finalStandardTokenizer src = newStandardTokenizer(getVersion(), reader);
    src.setMaxTokenLength(maxTokenLength);
    TokenStream tok = newStandardFilter(getVersion(), src);
    tok = newLowerCaseFilter(getVersion(), tok);
    tok = newStopFilter(getVersion(), tok, stopwords);
returnnew TokenStreamComponents(src, tok) {
@Override
protectedvoid setReader(final Reader reader) throws IOException {
        src.setMaxTokenLength(StandardAnalyzer.this.maxTokenLength);
super.setReader(reader);
      }
    };
  }

```

理解图：

![1541432164386](images/1541432164386.png)

Token就是分词过程中产生的对象（**包含分词的词语内容，该词在源文本中的开始和结束位置**）。

 

![1541432293057](images/1541432293057.png)



需要注意的是：**注意：搜索使用的分词器要和索引时使用的分词器一致。**





### 4.7 使用Luke查看索引

Luke作为Lucene工具包中的一个工具（http://www.getopt.org/luke/），可以通过界面来进行索引文件的查询、修改。

![1541432372830](images/1541432372830.png)



使用：

​	双击 start.bat

![1541432461849](images/1541432461849.png)

 

![1541432484348](images/1541432484348.png)





### 4.8 搜索流程

搜索的过程：

![1541432523011](images/1541432523011.png)

1、用户定义查询语句，用户确定查询什么内容（输入什么关键字）

2、IndexSearcher索引搜索对象，定义了很多搜索方法，程序员调用此方法搜索。

3、IndexReader索引读取对象，它对应的索引维护对象IndexWriter，IndexSearcher通过IndexReader读取索引目录中的索引文件

4、Directory索引流对象，IndexReader需要Directory读取索引库，使用FSDirectory文件系统流对象

5、IndexSearcher搜索完成，返回一个TopDocs（匹配度高的前边的一些记录）



#### 4.8.1 输入查询语句

	用户输入查询关键字,执行搜索之前需要先构建一个查询对象，查询对象中可以指定查询要搜索的Field文档域、查询关键字等，查询对象会生成具体的查询语法，
例如： 
语法 “name:lucene”表示要搜索Field域为name内容为“lucene”的文档

查询对象就相当于将查询的关键字封装好，封装了查询的语法。



#### 4.8.2 执行查询

传统方法是先找到文件，再找内容，如何在文件中找内容，在文件内容中匹配搜索关键字，这种方法是顺序扫描方法，数据量大就搜索慢。

倒排索引结构是根据内容（词语）找文档，倒排索引结构也叫反向索引结构，包括索引和文档两部分，索引即词汇表，它是在索引中匹配搜索关键字，由于索引内容量有限并且采用固定优化算法搜索速度很快，找到了索引中的词汇，词汇与文档关联，从而最终找到了文档。

![1541432625209](images/1541432625209.png)

比如：name:lucene    表示要从name域中查询lucene的文档，如下图查询出文档的ID列表

#### 4.8.3 渲染结果

(jsp)将查询的结果集进行处理，并渲染到页面，提供给用户一个友好的界面。参**考jd的搜索结果页面。**

#### 4.8.4 开发步骤



第一步：创建一个Directory对象，也就是索引库存放的位置。
第二步：创建一个indexReader对象，需要指定Directory对象。
第三步：创建一个indexsearcher对象，需要指定IndexReader对象
第四步：创建一个TermQuery对象，指定查询的域和查询的关键词。
第五步：执行查询。
第六步：返回查询结果。遍历查询结果并输出。
第七步：关闭IndexReader对象



```java
@Test
	public void doIndexSearch() throws IOException{
		//第一步骤：指定一个索引库的位置
		FSDirectory directory = FSDirectory.open(new File("G:\\index01"));
		//第二步：创建一个indexReader对象
		IndexReader reader = DirectoryReader.open(directory);
		//第三步：创建一个indexSearcher对象
		IndexSearcher searcher = new IndexSearcher(reader); 
		//第四步：创建一个termquery对象，指定查询的域和关键字
		TermQuery query = new TermQuery(new Term("name", "lucene"));
		//第五步：执行查询
		TopDocs topDocs = searcher.search(query, 10);
		//第六步：执行查询结果，遍历查询结果并输出
		System.out.println(">>>总记录数"+topDocs.totalHits);
		
		ScoreDoc[] docs = topDocs.scoreDocs;
		
		for (ScoreDoc scoreDoc : docs) {
			int id = scoreDoc.doc;//文档的id
			Document doc = searcher.doc(id);//获取doc对象
			
			System.out.println(doc.get("id"));
			System.out.println(doc.get("name"));
			System.out.println(doc.get("price"));
			System.out.println(doc.get("pic"));
			//System.out.println(doc.get("description"));		
		}
		//第七步：关闭资源
		reader.close();
	}

```



## 5 Field域

### 5.1 Field域属性

​	Field是文档中的域，包括Field名和Field值两部分，一个文档可以包括多个Field，Document只是Field的一个承载体，Field值即为要索引的内容，也是要搜索的内容。

+ 是否分词（tokenized） **分词的目的：为了索引**

  是，将field的内容分成一个一个单词。例如：**商品的名称，商品的介绍**

  否，不分词，将内容作为一个整体存储。 例如：商品ID 身份证号，图片路径

+ 是否索引（indexed） **索引的目的：为了搜索**

  是，将field的值建立索引。例如：**商品的名称**，商品的介绍

  否，不建立索引。例如：**图片路径**

+ 是否存储(stored)  **存储与否，要看页面是否需要展示**

  是,存储field的值。例如：商品名称，图片路径。

  否,不存储field的值。例如：商品介绍。如果需要展示，根据ID从数据库查询展示在详情页面。

### 5.2 Field常用的类型

| Field类      | 数据类型   | 是否分词 **tokenized** | 是否索引 indexed | 是否存储 stored | 说明        |
| ----------- | ------ | ------------------ | ------------ | ----------- | --------- |
| StringField | String | N                  | Y            | Y或者N        | 身份证号，商品ID |
| LongField   | Long   | Y                  | Y            | Y或者N        | 商品价格      |
| StoreField  | 多种类型   | N                  | N            | Y           | 图片路径      |
| TextField   | 字符串/流  | Y                  | Y            | Y或者N        | 商品名称，介绍   |



### 5.3 入门程序改造

图书id：
是否分词：不用分词，因为不会根据图书id来搜索图书
是否索引：索引，可能需要作为其他索引的查询条件
是否存储：要存储，因为查询结果页面需要使用id这个值。

图书名称：
是否分词：要分词，因为要将图书的名称内容分词索引，根据关键搜索图书名称抽取的词。
是否索引：要索引。
是否存储：要存储。

图书价格：
是否分词：要分词，lucene对数字型的值只要有搜索需求的都要分词和索引，因为lucene对数字型的内容要特殊分词处理，本例子可能要根据价格范围搜索，需要分词和索引。

是否索引：要索引
是否存储：要存储

图书图片地址：
是否分词：不分词
是否索引：不索引
是否存储：要存储

图书描述：
是否分词：要分词
是否索引：要索引
是否存储：因为图书描述内容量大，不在查询结果页面直接显示，不存储。

不存储是用做不在lucene的索引文件中记录，节省lucene的索引文件空间，如果要在详情页面显示描述，思路：
从lucene中取出图书的id，根据图书的id查询关系数据库中book表得到描述信息。

部分代码：

```java
document.add(new StringField("id",book.getId().toString(),Store.YES));//不分词，索引，要存储
document.add(new TextField("name",book.getName(),Store.YES));//分词，索引，存储
document.add(new FloatField("price",book.getPrice().floatValue(),Store.YES));//分词，索引，存储
document.add(new StoredField("pic",book.getPic()));//不分词，不索引，要存储
document.add(new TextField("description", book.getDescription(), Store.NO));//分词，索引，不存储

```

## 6.索引维护

lucene中操作索引和文档相关也涉及到CRUD操作。而比较关键的就是搜索。例如使用搜索流程实现JD的搜索功能等。

### 6.1索引添加

​	参考索引流程代码。

### 6.2索引更新

​	更新索引是先删除再添加，如果没有查到文档，直接添加。建议对更新需求采用此方法并且要保证对已存在的索引执行更新，可以先查询出来，确定更新记录存在，再执行更新操作。

代码如下：

```java
/**
	 * 更新  是先删除再添加。如果没有文档，就直接添加。
	 * @throws Exception
	 */
	@Test
	public void update() throws Exception {
		// 获取indexWriter对象
		IndexWriter indexwriter = getIndexwriter();
		//先构建更新后的文档
		Document document = new Document();
		document.add(new TextField("name", "lucene", Store.YES));
		//查询出name为lucne的文档，再将该文档更新成新的。如果不存在直接添加文档   已经验证过了。
		indexwriter.updateDocument(new Term("name","java"), document);
		//关闭流
		indexwriter.close();
	}

```



### 6.3索引删除

+ 根据查询结果删除

```java
	// 删除索引
	@Test
	public void deleteIndex() throws Exception {
		// 1、指定索引库目录
		Directory directory = FSDirectory.open(new File("E:\\11-index\\0720"));
		// 2、创建IndexWriterConfig
		IndexWriterConfig cfg = new IndexWriterConfig(Version.LATEST,
				new StandardAnalyzer());
		// 3、创建IndexWriter
		IndexWriter writer = new IndexWriter(directory, cfg);
		// 4、通过IndexWriter来删除索引
		// b)、删除指定索引
		writer.deleteDocuments(new Term("filename", "apache"));
		// 5、关闭IndexWriter
		writer.close();
	}

```

+ 删除所有

```java
// 删除索引
	@Test
	public void deleteIndex() throws Exception {
		// 1、指定索引库目录
		Directory directory = FSDirectory.open(new File("E:\\11-index\\0720"));
		// 2、创建IndexWriterConfig
		IndexWriterConfig cfg = new IndexWriterConfig(Version.LATEST,
				new StandardAnalyzer());
		// 3、创建IndexWriter
		IndexWriter writer = new IndexWriter(directory, cfg);
		// 4、通过IndexWriter来删除索引
		// a)、删除全部索引
		writer.deleteAll();
		// 5、关闭IndexWriter
		writer.close();
	}

```



## 7.中文分词器

### 7.1 什么是中文分词器

​	学过英文的都知道，英文是以单词为单位的，单词与单词之间以空格或者逗号句号隔开。而中文则以字为单位，字又组成词，字和词再组成句子。所以对于英文，我们可以简单以空格判断某个字符串是否为一个单词，比如I love China，love 和 China很容易被程序区分开来；但中文“我爱中国”就不一样了，电脑不知道“中国”是一个词语还是“爱中”是一个词语。把中文的句子切分成有意义的词，就是中文分词，也称切词。我爱中国，分词的结果是：我爱中国。

综上，以目前的方式分词中文，不能达到我们的要求，这时需要用到分词中文的分词器，也就是中文分词器。

### 7.2 Lucene自带的中文分词器

+ StandardAnalyzer  这个是对中文一个一个拆分。如：“我爱中国”；效果：“我”、“爱”、“中”、“国”

  ​

+ CJKAnalyzer  这个对中文两个两个拆分 。如：“我是中国人”，效果：“我是”、“是中”、“中国”、“国人”。

  ​

以上两种都不适合，达不到要求。

+ paoding

```
庖丁解牛最新版在 https://code.google.com/p/paoding/ 中最多支持Lucene 3.0，且最新提交的代码在 2008-06-03，在svn中最新也是2010年提交，已经过时，不予考虑。
```

+ mmseg4j

```
最新版已从 https://code.google.com/p/mmseg4j/ 移至 https://github.com/chenlb/mmseg4j-solr，支持Lucene 4.10，且在github中最新提交代码是2014年6月，从09年～14年一共有：18个版本，也就是一年几乎有3个大小版本，有较大的活跃度，用了mmseg算法。
```

+ IKAnalyzer

```
最新版在https://code.google.com/p/ik-analyzer/上，支持Lucene 4.10从2006年12月推出1.0版开始， IKAnalyzer已经推出了4个大版本。最初，它是以开源项目Luence为应用主体的，结合词典分词和文法分析算法的中文分词组件。从3.0版本开 始，IK发展为面向Java的公用分词组件，独立于Lucene项目，同时提供了对Lucene的默认优化实现。在2012版本中，IK实现了简单的分词 歧义排除算法，标志着IK分词器从单纯的词典分词向模拟语义分词衍化。 但是也就是2012年12月后没有在更新。
综合来说：它重要的特性就是：第一：可以自定义停用词 第二：可以自定义扩展词。
```

+ jcseg

```
最新版本在git.oschina.net/lionsoul/jcseg，支持Lucene 4.10，作者有较高的活跃度。利用mmseg算法
```



推荐使用IKAnalyzer.



### 7.3 IKAnalyzer

​	IKAnalyzer继承Lucene的Analyzer抽象类，使用IKAnalyzer和Lucene自带的分词器方法一样，将Analyzer测试代码改为IKAnalyzer测试中文分词效果.

注意：

​	如果使用了IK分词器 来进行分词 ，那么在搜索的时候也应当需要使用它，否则会出现查询结果不一致的情况。

![1541482597813](images/1541482597813.png)



#### 7.3.1 添加依赖



```xml
<!--第三方ik分词器-->
<dependency>
    <groupId>com.janeluo</groupId>
    <artifactId>ikanalyzer</artifactId>
    <version>2012_u6</version>
    <exclusions>
        <exclusion>
            <artifactId>lucene-analyzers-common</artifactId>
            <groupId>org.apache.lucene</groupId>
        </exclusion>
        <exclusion>
            <artifactId>lucene-core</artifactId>
            <groupId>org.apache.lucene</groupId>
        </exclusion>
        <exclusion>
            <artifactId>lucene-queryparser</artifactId>
            <groupId>org.apache.lucene</groupId>
        </exclusion>
    </exclusions>
</dependency>

```



#### 7.3.2 添加配置文件

解压zip包，并将解压后的三个配置文件（mydict.dic,IKAnalyzer.cfg.xml,ext_stopword.dic） 拷贝到项目工程中classpath 下

![1541483026641](images/1541483026641.png)

IKAnalyzer.cfg.xml配置文件配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典--> 
	<entry key="ext_dict">mydict.dic;</entry> 
	 
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">ext_stopword.dic</entry> 
</properties>

```

#### 7.3.3 使用IK

修改创建索引时使用的分词器

```java
// 创建中文分词器
Analyzer analyzer = new IKAnalyzer();

```



扩展中文词库内容：

在my_dic.dic文件中添加 【传智播客】四个字用于测试作为一个整体词。

在ext_stopword.dic 中添加【国民党】三个字用于测试 作为过滤词过滤



注意：在编辑词典的时候，切记，

1.词典的编码格式一定是不带BOM的UTF-8格式

2.不要使用记事本打开，否则就会带有BOM



测试分词效果：



```java
@Test
public void testAnalyzer() throws IOException {
    //1.创建analyzer (ik)
    Analyzer analyzer = new IKAnalyzer();
    //2.获取tokenstream  (分的词都在此对象中)
    //第一个参数：就是域的名称，可以不写或者""
    //第二个参数：分析的词内容
    TokenStream tokenStream = analyzer.tokenStream("", "共产党是一个伟大的党啊");
    //3.指定一个引用   （指定 词的引用   或者 偏移量）
    CharTermAttribute addAttribute = tokenStream.addAttribute(CharTermAttribute.class);
    //4.设置一个偏移量的引用
    OffsetAttribute offsetAttribute = tokenStream.addAttribute(OffsetAttribute.class);
    //5.调用tokenstream的rest方法 重置
    tokenStream.reset();
    //6.通过wihle 循环 遍历单词列表
    while(tokenStream.incrementToken()){
        ///打印
        System.out.println("start>>"+offsetAttribute.startOffset());
        System.out.println(addAttribute.toString());//打印单词
        System.out.println("end>>"+offsetAttribute.endOffset());
    }
    tokenStream.close();
    //关闭流
}


```

lucene搜索时有自己的语法，简单说明如下：



![1541483642242](images/1541483642242.png)













