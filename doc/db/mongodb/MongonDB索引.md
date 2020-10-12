&emsp;&emsp;**索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中（ 索引存储在特定字段或字段集的值）**，而且是**使用了B-tree结构**。索引可以极大程度提升MongoDB查询效率。  
&emsp;&emsp;如果没有索引，MongoDB必须执行全集合collections扫描，即扫描集合中的每个文档，选取符合查询条件的文档document。 如果查询时存在适当的索引，MongoDB可以使用索引来限制它必须查询的文档document的数量，特别是在处理大量数据时，所以**选择正确的索引是很关键的、重要的**。  

**创建索引，需要考虑的问题：**  
+ **每个索引至少需要数据空间为8kb；**
+ **添加索引会对写入操作会产生一些性能影响。 对于具有高写入率的集合Collections，索引很昂贵，因为每个插入也必须更新任何索引；**
+ **索引对于具有高读取率的集合Collections很有利，不会影响没索引查询；**
+ **处于索引处于action状态时，每个索引都会占用磁盘空间和内存，因此需要对这种情况进行跟踪检测。**

**索引限制：**
+ **索引名称长度不能超过128字段；**
+ **复合索引不能超过32个属性；**
+ **每个集合Collection不能超过64个索引；**
+ **不同类型索引还具有各自的限制条件。**

## 1. 索引管理
### 1.1 索引创建
**索引创建使用createIndex()方法，格式如下:**

    db.collection.createIndex(<key and index type specification>,<options>)
**createIndex() 接收可选参数，可选参数列表如下：**
Parameter|Type|Description
:-|:-|:-|
background	|Boolean	|建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为false。
unique	|Boolean|	建立的索引是否唯一。指定为true创建唯一索引。默认值为false.
name|	string	|索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。
dropDups|	Boolean	|3.0+版本已废弃。在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 false.
sparse|	Boolean|	对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 false.
expireAfterSeconds|	integer|	指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。
v|	index version|	索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。
weights|	document|	索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。
default_language|	string	|对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语
language_override|	string	|对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language.

### 1.2 查看索引
**查看Collection中所有索引，格式如下:**   

    db.collection.getIndexes()

### 1.3 删除索引
**删除Collection中的索引：格式如下:**  

	db.collection.dropIndexes()   //删除所有索引
	db.collection.dropIndex()    //删除指定的索引  
### 1.4 索引名称
索引的默认名称是索引键和索引中每个键的value1或-1，形式index_name+1/-1，比如：

    db.products.createIndex( { item: 1, quantity: -1 } )----索引名称为item_1_quantity_-1
也可以指定索引名称：

    db.products.createIndex( { item: 1, quantity: -1 } , { name: "inventory" } )  ----索引名称为inventory
### 1.5 查看索引创建过程以及终止索引创建

方法|解析
:-|:-|
db.currentOp()|查看索引创建过程
db.killOp(opid)|终止索引创建，其中-opid为操作id
    
### 1.6 索引使用情况
形式|解析
:-|:-|
\$indexStats|获取索引访问信息
explain()|返回查询情况：在executionStats模式下使用db.collection.explain（）或cursor.explain（）方法返回有关查询过程的统计信息，包括使用的索引，扫描的文档数以及查询处理的时间（以毫秒为单位）。
Hint()|控制索引，例如要强制MongoDB使用特定索引进行db.collection.find（）操作，请使用hint（）方法指定索引
### 1.7 MongoDB度量标准
MongoDB提供了许多索引使用和操作的度量标准，在分析数据库的索引使用时可能需要考虑这些度量标准，如下所示：

形式|解析
:-|:-|
metrics.queryExecutor.scanned|在查询和查询计划评估期间扫描的索引项的总数
metrics.operation.scanAndOrder|返回无法使用索引执行排序操作的已排序数字的查询总数
collStats.totalIndexSize|所有索引的总大小。 scale参数会影响此值。如果索引使用前缀压缩（这是WiredTiger的默认值），则返回的大小将反映计算总计时任何此类索引的压缩大小。
collStats.indexSizes|指定集合collection上每个现有索引的键和大小。 scale参数会影响此值
dbStats.indexes|包含数据库中所有集合的索引总数的计数。
dbStats.indexSize|在此数据库上创建的所有索引的总大小

### 1.8 后台索引操作
&emsp;&emsp;在密集(快达到数据库最大容量）Collection创建索引：在默认情况下，在密集的Collection（快达到数据库最大容量）时创建索引，会阻止其他操作。在给密集的Collection（快达到数据库最大容量）创建索引时，
索引构建完成之前，保存Collection的数据库不可用于读取或写入操作。 任何需要对所有数据库（例如listDatabases）进行读或写锁定的操作都将等待不是后台进程的索引构建完成。  

因此**可以使用background属性进行设置后台索引创建**，操作如下：

    db.people.createIndex( { zipcode: 1 }, { background: true } )
    默认情况下，在创建索引时，background为false，可以和其他属性进行组合使用：
    db.people.createIndex( { zipcode: 1 }, { background: true, sparse: true } )
## 2. 索引类型
### 2.1 单字段索引(Single Field Indexes)
&emsp;&emsp;**MongoDB可以在任何一个字段中创建索引，默认情况下，所有的集合(collections)会在_id字段中创建索引。**_id索引是为防止客户端插入具有相同value的_id字段的文档Document，而且不能删除_id字段索引。  
&emsp;&emsp;**在分片群集中使用_id索引，如果不使用_id字段作为分片键，则应用程序必须确保_id字段中值的唯一性以防止出错，解决方法为使用标准的自动生成的ObjectId来完成。**   
&emsp;&emsp;**一般单字段索引的value中，“1”指定按升序对项目进行排序的索引，“-1”指定按降序对项目进行排序的索引。如下所示：**  

![](https://user-gold-cdn.xitu.io/2019/8/3/16c54d09036b0552?w=773&h=300&f=png&s=13236)

**在单个字段创建索引，示例如下：**   

    {
      "_id": ObjectId("570c04a4ad233577f97dc459"),
      "score": 1034,
      "location": { state: "NY", city: "New York" }
    }
    //创建单字段索引
     db.records.createIndex( { score: 1 } )
     //支持的查询  
    db.records.find( { score: 2 } )
    db.records.find( { score: { $gt: 10 } } )

**在嵌入式文档Document中的字段创建索引，示例如下：**  

    db.records.createIndex( { "location.state": 1 } )
    //支持的查询 
    db.records.find( { "location.state": "CA" } )
    db.records.find( { "location.city": "Albany", "location.state": "NY" } )
**在嵌入式文档Document创建索引，示例如下：**  

    db.records.createIndex( { location: 1 } )
    //支持查询 
    db.records.find( { location: { city: "New York", state: "NY" } } )
  
  
### 2.2 复合索引(Compound Index)
**复合索引**指的是将多个key组合到一起创建索引，这样可以加速匹配多个键的查询。特性如下：
+ **MongoDB对任何复合索引都限制了32个字段;**
+ **无法创建具有散列索引(hash index)类型的复合索引。如果尝试创建包含散列索引字段的复合索引，则会报错;**
+ **复合索引创建字段索引的顺序是很重要的。因为索引以升序（1）或降序（-1）排序顺序存储对字段的引用; 对于单字段索引，键的排序顺序无关紧要，因为MongoDB可以在任一方向上遍历索引。 但是，对于复合索引，排序顺序可以决定索引是否可以支持排序操作;**
+ **除了支持在所有索引字段上匹配的查询之外，复合索引还可以支持与索引字段的前缀匹配的查询。**

**创建复合索引的格式：**  

    db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )

**排序顺序，两个字段的复合索引示例，index{userid：1，score：-1}，先userid的value排序，然后再userid排序基础下进行score排序。如下图：** 
![](https://user-gold-cdn.xitu.io/2019/8/3/16c55a2716c1ab78?w=751&h=311&f=png&s=17991)![](https://user-gold-cdn.xitu.io/2019/8/3/16c55a2716c1ab78?w=751&h=311&f=png&s=17991)


**创建复合索引，示例如下：**  
    
    {
     "_id": ObjectId(...),
     "item": "Banana",
     "category": ["food", "produce", "grocery"],
     "location": "4th Street Store",
     "stock": 4,
     "type": "cases"
    }
    //创建复合索引
    db.products.createIndex( { "item": 1, "stock": 1 } )
    //支持的查询
    db.products.find( { item: "Banana" } )
    db.products.find( { item: "Banana", stock: { $gt: 5 } } )

**复合索引中的前缀查询，示例如下：**  
    
    //创建复合索引
    db.products.createIndex({ "item": 1, "location": 1, "stock": 1 })
    //前缀为：{ item: 1 }与{ item: 1, location: 1 }
    //支持前缀查询为   
     db.products.find( { item: "Banana" } )
     db.products.find( { item: "Banana", location: “beijing”} )
    //不支持前缀查询，不会提高查询效率
    //不包含前缀字段
     db.products.find( { location: “beijing”} )
     db.products.find( { stock: { $gt: 5 } )
     db.products.find( { location: “beijing”，stock: { $gt: 5 } )
     //不按照创建复合索引字段顺序的前缀查询
     db.products.find( { location: “beijing”，item: "Banana" },stock: { $gt: 5 } )

### 2.3 多键索引
&emsp;&emsp;**MongoDB使用多键索引为数组的每个元素都创建索引，多键索引可以建立在字符串、数字等key或者内嵌文档(document)的数组上，如果索引字段包含数组值，MongoDB会自动确定是否创建多键索引; 您不需要手动指定多键类型。** 其中创建方式：  

    db.coll.createIndex( { <field>: < 1 or -1 > } )

**索引边界**  
&emsp;&emsp;使用多键索引，会出现索引边界（索引边界即是查询过程中索引能查找的范围）的计算，并计算必须遵循一些规则。即当多个查询的条件中字段都存在索引中时，MongoDB将会使用交集或者并集等来判断这些条件索引字段的边界最终产生一个最小的查找范围。可以分情况：  
**1).交集边界**  
&emsp;&emsp;**交集边界即为多个边界的逻辑交集**，对于给定的数组字段，假定一个查询使用了数组的多个条件字段并且可以使用多键索引。**如果使用了$elemMatch连接了条件字段，则MongoDB将会相交多键索引边界**，示例如下：  

    //survey Collection中document有一个item字段和一个ratings数组字段
    { _id: 1, item: "ABC", ratings: [ 2, 9 ] } 
    { _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
    
    //在ratings数组上创建多键索引： 
	db.survey.createIndex({ratings:1})
	
    //两种查询
	db.survey.find({ratings:{$elemMatch:{$gte:3,$lte:6}}})  //(1)
	db.survey.find( { ratings : { $gte: 3, $lte: 6 } } )  //(2)
&emsp;&emsp;**查询条件分别为大于等于3、小于等于6**，其中 **(1)中使用了\$elemMatch连接查询条件，会产生一个交集ratings：[[3,6]。在(2)查询中，没使用\$elemMatch,则不会产生交集，只要满足任何一个条件即可。**

**2).并集边界**  
&emsp;&emsp;**并集边界常常用在确定多键组合索引的边界**，例如：给定的组合索引{a:1,b:1}，在字段a上有一个边界：[3，+∞），在字段b上有一个边界：（-∞，6]，相并这两个边界的结果是：{ a: [ [ 3, Infinity ] ], b: [ [ -Infinity, 6 ] ] }。

&emsp;&emsp;**而且如果MongoDB没法并集这两个边界，MongoDB将会强制使用索引的第一个字段的边界来进行索引扫描，在这种情况下就是： a: [ [ 3, Infinity ] ]。**  

**3、数组字段的组合索引**  
&emsp;&emsp;一个组合索引的索引字段是数组，例如一个survey collection集合document文档中含有item字段和ratings数组字段，示例如下：  

    { _id: 1, item: "ABC", ratings: [ 2, 9 ] } 
    { _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
    
    //在item字段和ratings字段创建一个组合索引：
	db.survey.createIndex( { item: 1, ratings: 1 } )
	
	//查询条件索引包含的两个key
	db.survey.find( { item: "XYZ", ratings: { $gte: 3 } } )
**分别处理查询条件：**
	
	item: "XYZ" -->  [ [ "XYZ", "XYZ" ] ];
	ratings: { $gte: 3 } -->  [ [ 3, Infinity ] ].
**MongoDB使用并集边界来组合这两个边界：** 

	{ item: [ [ "XYZ", "XYZ" ] ], ratings: [ [ 3, Infinity ] ] }  

**4).内嵌文档document的数组上建立组合索引**  
&emsp;&emsp;如果数组中包含内嵌文档document，想在包含的内嵌文档document字段上建立索引，需要在索引声明中使用**逗号“,”** 来分隔字段名,示例如下：

    ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]
则score字段名称就是：ratings.score。

**5).混合不是数组类型的字段和数组类型字段的并集**  

    { _id: 1, item: "ABC", ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ] } 
    { _id: 2, item: "XYZ", ratings: [ { score: 5, by: "anon" }, { score: 7, by: "wv" } ] }
    
    //在item和数组字段ratings.score和ratings.by上创建一个组合索引
    db.survey2.createIndex( { "item": 1, "ratings.score": 1, "ratings.by": 1 } )
    
    //查询
    db.survey2.find( { item: "XYZ", "ratings.score": { $lte: 5 }, "ratings.by": "anon" } )
    
**分别对查询条件进行处理：**
	
	item: "XYZ"--> [ ["XYZ","XYZ"] ];
	score: {$lte:5}--> [[-Infinity,5]];
	by: "anon" -->["anon","anon"].
&emsp;&emsp;MongoDB可以组合 item键的边界与 ratings.score和ratings.by两个边界中的一个，到底是score还是by索引边界这取决于查询条件和索引键的值。MongoDB不能确保哪个边界和item字段进行并集。	
但如果想**组合ratings.score和ratings.by边界**，则**查询必须使用\$elemMatch**。

**6).数组字段索引的并集边界**  
&emsp;&emsp;在数组内部并集索引键的边界,
+ 除了字段名称外，索引键必须有相同的字段路径,
+ 查询的时候必须在路径上使用$elemMatch进行声明
+ 对于内嵌的文档，使用逗号分隔的路径,比如a.b.c.d是字段d的路径。为了在相同的数组上并集索引键的边界，需要$elemMatch必须使用在a.b.c的路径上。  

**比如：在ratings.score和ratings.by字段上创建组合索引：**  

    db.survey2.createIndex( { "ratings.score": 1, "ratings.by": 1 } )
**字段ratings.score和ratings.by拥有共同的路径ratings。下面的查询使用$elemMatch则要求ratings字段必须包含一个元素匹配这两个条件：**
	
    db.survey2.find( { ratings: { $elemMatch: { score: { $lte: 5 }, by: "anon" } } } )
**分别对查询条件进行处理：**
	
	score: { $lte: 5 } --> [ -Infinity, 5 ];
	by: "anon"--> [ "anon", "anon" ].
**MongoDB可以使用并集边界来组合这两个边界：**
	
	{ "ratings.score" : [ [ -Infinity, 5 ] ], "ratings.by" : [ [ "anon", "anon" ] ] }  
	
**7). 还有不使用\$elemMatch进行查询以及不完整的路径上使用\$elemMatch，想要了解更多可以查看《[官方文档-Multikey Index Bounds](https://docs.mongodb.com/manual/core/multikey-index-bounds/)》。**   

<br>

**限制：**
+ **对于一个组合多键索引，每个索引文档最多只能有一个索引字段的值是数组。如果组合多键索引已经存在了，不能在插入文档的时候违反这个限制;**
+ **不能声明一个多键索引作为分片键索引;**
+ **哈希索引不能拥有多键索引;**
+ **多键索引不能进行覆盖查询;**
+ **当一个查询声明把数组整体作为精确匹配的时候，MongoDB可以使用多键索引来查找这个查询数组的第一个元素，但是不能使用多键索引扫描来找出整个数组。代替方案是当使用多键索引查询出数组的第一个元素之后，MongoDB再对过滤之后的文档再进行一次数组匹配。**

### 2.4 全文索引(text index)
&emsp;&emsp;MongoDB提供了一种**全文索引类型，支持在Collection中搜索字符串内容，对字符串与字符串数组创建全文可搜索的索引**
。 这些全文索引不存储特定于语言的停用词（例如“the”，“a”，“或”），并且阻止document集合中的单词仅存储根词。创建方式如下：

    db.collection.createIndex( { key: "text"，key："text" ..... } )
**而且MongoDB提供权重以及通配符的创建方式。查询方式多个字符串空格隔开，排除查询使用“-”如下所示：**  

    db.collection.find({$text:{$search:"runoob add -cc"}})
要删除全本索引，需要将**索引的名称**传递给**db.collection.dropIndex()方法**， 而要获取索引的名称，使用db.collection.getIndexes（）方法。

&emsp;&emsp;还可以指定全文索引的语言，通过**default_language属性** 在创建时指定， 或者使用**language_override属性** 覆盖掉创建document文档时默认的语言，如下所示：

    //指定不同的语言的方法：创建全文索引的时候使用default_language属性
    db.collection.createIndex(
      { content : "text" },
      { default_language: "spanish" })
      
    //使用language_override属性覆盖默认的语言
    db.quotes.createIndex( { quote : "text" },
                       { language_override: "idioma" } )
                       
    //默认的全文索引名称为context_text,users.comments.text，指定名称MyTextIndex
    db.collection.createIndex(
       {
         content: "text",
         "users.comments": "text",
         "users.profiles": "text"
       },
       {
         name: "MyTextIndex"
       }
    )
**权重**  
&emsp;&emsp;**每个全文索引可以通过设置权重来分配不同的搜索程度，默认权重为1，对于文档中的每个索引字段，MongoDB将匹配数乘以权重并将结果相加。 使用此总和，MongoDB然后计算文档的分数，示例如下：**

    {
      _id: 1,
      content: "This morning I had a cup of coffee.",
      about: "beverage",
      keywords: [ "coffee" ]
    }
    {
      _id: 2,
      content: "Who doesn't like cake?",
      about: "food",
      keywords: [ "cake", "food", "dessert" ]
    }
    
    //通过db.blog.createIndex来指定weight权重
    db.blog.createIndex(
       {
         content: "text",
         keywords: "text",
         about: "text"
       },
       {
         weights: {
           content: 10,
           keywords: 5
         },
         name: "TextIndex"
       }
     )
**content权重为10，keywords为5，about为默认权重1，因此可以得出content对于keywords查询频率高于2倍，而对于about字段则是10倍。**

**通配符全文索引**  
&emsp;&emsp;**在多个字段上创建全文索引时，还可以使用通配符说明符（$\**）**。 使用通配符全文索引，MongoDB会为包含Collection中每个Document的字符串数据。例如： 

    db.collection.createIndex( { "$**": "text" } )
通配符全本索引是多个字段上的全本索引。 因此，可以在创建索引期间为特定字段指定权重，以控制结果的排名。

**限制**
+ **每个Collection一个全文索引：一个collection最多只有一个全文索引，**
+ **Text Search 和Hints函数，如果查询包含$ text查询表达式，则不能使用hint（）；**
+ **Text Index and Sort，排序操作无法从文本索引获取排序顺序，即使是复合文本索引也是如此; 即排序操作不能使用文本索引中的排序；**
+ **复合索引：复合索引可以包括文本索引键与升序/降序索引键的组合。 但是，这些复合索引具有以下限制：  
	1).复合文本索引不能包含任何其他特殊索引类型，例如多键或地理空间索引字段。  
	2).如果复合文本索引包括文本索引键之前的键，则执行$ text搜索时，查询谓词必须包含前面键上的相等匹配条件。  
	3).创建复合文本索引时，必须在索引规范文档中相邻地列出所有文本索引键。**

### 2.5 Hash 索引
&emsp;&emsp;**散列索引使用散列函数来计算索引字段值的散列值。** 散列函数会折叠嵌入的文档并计算整个值的散列值，但不支持多键（即数组）索引。
生成hash索引key使用了convertShardKeyToHashed()方法。创建方式如下：

    db.collection.createIndex( { _id: "hashed" } )
**而且散列索引支持使用散列分片键进行分片。 基于散列的分片使用字段的散列索引作为分片键来分割整个分片群集中的数据。**
## 3. 索引属性
**索引属性有TTL索引、惟一性索引、部分索引、稀疏索引以及区分大小写索引。**
### 3.1 TTL索引(TTL Indexes)
&emsp;&emsp;**TTL索引**是特殊的单字段索引，并且**字段类型**必须是**date类型或者包含有date类型的数组**，MongoDB可以使用它在一定时间后或在特定时钟时间自动从集合中删除文档。 数据到期对于某些类型的信息非常有用，例如机器生成的事件数据，日志和会话信息，这些信息只需要在数据库中持续有限的时间。

**创建TTL索引方法，和普通索引的创建方法一样，只是会多加一个expireAfterSeconds的属性，格式如下：**  

    db.collection.createIndex( {key and index type specification},{ expireAfterSeconds: time})
**例子：** 

    db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )

**指定过期时间**   
**首先在保存BSON日期类型值或BSON日期类型对象数组的字段上创建TTL索引，并指定expireAfterSeconds值为0.对于集合中的每个文档，设置 索引日期字段为与文档到期时间对应的值。示例操作如下：**  
**第一步：** 

    db.log_events.createIndex( { "expireAt": 1 }, { expireAfterSeconds: 0 } )
**第二步：** 

    db.log_events.insert( {
       "expireAt": new Date('July 22, 2013 14:00:00'),
       "logEvent": 2,
       "logMessage": "Success!"
    } )


**数据过期类型：**
+ **当指定时间到了过期的阈值数据就会过期并删除；**
+ **如果字段是数组，并且索引中有多个日期值，MongoDB使用数组中最低（即最早）的日期值来计算到期阈值；**
+ **如果文档(document)中的索引字段不是日期或包含日期值的数组，则文档(document)将不会过期；**
+ **如果文档(document)不包含索引字段，则文档(document)不会过期。**

**TTL索引特有限制：**
+ **TTL索引是单字段索引。 复合索引不支持TTL并忽略expireAfterSeconds选项；**
+ **_id属性不支持TTL索引；**
+ **无法在上限集合上创建TTL索引，因为MongoDB无法从上限集合中删除文档；**
+ **不能使用createIndex()方法来更改现有索引的expireAfterSeconds值。而是将collMod数据库命令与索引集合标志结合使用。 否则，要更改现有索引的选项的值，必须先删除索引并重新创建；**
+ **如果字段已存在非TTL单字段索引，则无法在同一字段上创建TTL索引，因为无法在相同key创建不同类型的的索引。 要将非TTL单字段索引更改为TTL索引，必须先删除索引并使用expireAfterSeconds选项重新创建。**

### 3.2 惟一性索引(Unique Indexes)
&emsp;&emsp;**唯一索引**可确保索引字段不存储重复值; 即强制索引字段的唯一性。 默认情况下，MongoDB在创建集合期间在_id字段上创建唯一索引。创建方式如下：

    db.collection.createIndex( <key and index type specification>, { unique: true } )
**单个字段创建方式，示例如下：**

    db.members.createIndex( { "user_id": 1 }, { unique: true } )
**唯一性复合索引：** 还可以对复合索引强制执行唯一约束。 如果对复合索引使用唯一约束，则MongoDB将对索引键值的组合强制实施唯一性。示例如下：
    
    //创建的索引且强制groupNumber，lastname和firstname值组合的唯一性。
    db.members.createIndex( { groupNumber: 1, lastname: 1, firstname: 1 }, { unique: true } )
**唯一多键索引：**   

    { _id: 1, a: [ { loc: "A", qty: 5 }, { qty: 10 } ] }
    
    //创建索引：
    db.collection.createIndex( { "a.loc": 1, "a.qty": 1 }, { unique: true } )
    
    //插入数据：唯一索引允许将以下Document插入Collection中，因为索引强制执行a.loc和a.qty值组合的唯一性：
    db.collection.insert( { _id: 2, a: [ { loc: "A" }, { qty: 5 } ] } )
    db.collection.insert( { _id: 3, a: [ { loc: "A", qty: 10 } ] } )
**创建唯一索引到副本或者分片中：**
对于副本集和分片集群，使用滚动过程创建唯一索引需要在过程中停止对集合的所有写入。 如果在过程中无法停止对集合的所有写入，请不要使用滚动过程。 相反，通过以下方式在集合上构建唯一索引： 
+ 在主服务器上为副本集发出db.collection.createIndex（）
+ 在mongos上为分片群集发出db.collection.createIndex（）   

**NOTE：详细解析可以看**

**限制：**
+ **如果集合已经包含超出索引的唯一约束的数据（即有重复数据），则MongoDB无法在指定的索引字段上创建唯一索引。**
+ **不能在hash索引上创建唯一索引**
+ **唯一约束适用于Collection中的一个Document。由于约束适用于单文档document，因此对于唯一的多键索引，只要该文档document的索引键值不与另一个文档document的索引键值重复，文档就可能具有导致重复索引键值的数组元素。 在这种情况下，重复索引记录仅插入索引一次。**
+ **分片Collection唯一索引只能如下：  
    1).分片键上的索引  
	2).分片键是前缀的复合索引  
    3).	默认的_id索引; 但是，如果_id字段不是分片键或分片键的前缀，则_id索引仅对每个分片强制执行唯一性约束。如果_id字段不是分片键，也不是分片键的前缀，MongoDB希望应用程序在分片中强制执行_id值的唯一性。**

### 3.3 部分索引(Partial Indexes)
&emsp;&emsp;**部分索引通过指定的过滤表达式去达到局部搜索**。通过db.collection.createIndex()方法中增加partialFilterExpression属性创建，过滤表达式如下：
+ **等式表达式（即 file：value或使用\$eq运算符）**
+ **\$exists表达式**
+ **\$gt，\$gte，\$lt，\$lte 表达式**
+ **\$type表达式**
+ **\$and**

**示例如下**： 

    //创建部分索引
    db.restaurants.createIndex(
       { cuisine: 1, name: 1 },
       { partialFilterExpression: { rating: { $gt: 5 } } }
    )
    //查询情况分类
    db.restaurants.find( { cuisine: "Italian", rating: { $gte: 8 } } )   //(1)
    db.restaurants.find( { cuisine: "Italian", rating: { $lt: 8 } } )    //(2)
    db.restaurants.find( { cuisine: "Italian" } )                        //(3)
其中：
+ **(1)查询：** 查询条件{ \$gte: 8 }于创建索引条件{ \$gt: 5 }可以构成一个完整集(**查询条件是创建索引条件的子集，即大于5可以包含大于等于 8**),可以使用部分索引查询。  
+ **(2)查询：** 条件达不到完整集，MongoDB将不会将部分索引用于查询或排序操作。  
+ **(3)查询：** 次查询**没有使用过滤表达式**，也不会使用部分索引，因为要使用部分索引，查询必须包含过滤器表达式（或指定过滤器表达式子集的已修改过滤器表达式）作为其查询条件的一部分 

**限制：**
+ **不可以仅通过过滤表达式创建多个局部索引；**
+ **不可以同时使用局部索引和稀疏索引（sparse index）；**
+ **_id索引不能使用局部索引，分片索引（shard key index）也不能使用局部索引；**
+ **同时指定partialFilterExpression和唯一约束，则唯一约束仅适用于满足过滤器表达式的文档。 如果Document不符合筛选条件，则具有唯一约束的部分索引是允许插入不符合唯一约束的Document。**

### 3.4 稀疏索引(Sparse Indexes)
&emsp;&emsp;稀疏索只引搜索包含有索引字段的文档的条目，跳过索引键不存在的文档，即稀疏索引不会搜索不包含稀疏索引的文档。默认情况下， 2dsphere (version 2), 2d, geoHaystack, 全文索引等总是稀疏索引。创建方式db.collection.createIndex()方法增加sparse属性，如下所示：

    db.addresses.createIndex( { "xmpp_id": 1 }, { sparse: true } )

**稀疏索引不被使用的情况:**  如果稀疏索引会导致查询和排序操作的结果集不完整，MongoDB将不会使用该索引，除非hint()示显式指定索引。

**稀疏复合索引：**
+ 对于包含上升/下降排序的稀疏复合索引，只要复合索引中的一个key 索引存在都会被检测出来
+ 对于包含上升/下降排序的包含地理空间可以的稀疏复合索引，只有存在地理空间key才能被检测出来
+ 对于包含上升/下降排序的全文索引的稀疏复合索引，只有存在全文索引索引才可以被检测

**稀疏索引与唯一性：** 一个既包含稀疏又包含唯一的索引避免集合上存在一些重复值得文档，但是允许多个文档忽略该键。满足稀疏索引和唯一性操作其两个限制都要遵循。

**整合示例如下：**

    { "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
    { "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
    { "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
  
    //在score中创建稀疏索引： 
    db.scores.createIndex( { score: 1 } , { sparse: true } )
    
    //查询
    db.scores.find( { score: { $lt: 90 } } )    //(1)
    db.scores.find().sort( { score: -1 } )      //(2)
    db.scores.find().sort( { score: -1 } ).hint( { score: 1 } ) //(3)
  
    //在score字段上创建具有唯一约束和稀疏过滤器的索引：
    db.scores.createIndex( { score: 1 } , { sparse: true, unique: true } )
    
    //该索引允许插入具有score字段的唯一值的文档或不包括得分字段的文档。如下：
    db.scores.insert( { "userid": "AAAAAAA", "score": 43 } )
    db.scores.insert( { "userid": "BBBBBBB", "score": 34 } )
    db.scores.insert( { "userid": "CCCCCCC" } )
    db.scores.insert( { "userid": "DDDDDDD" } )
    
    //索引不允许添加以下文档，因为已存在score值为82和90的文档
    db.scores.insert( { "userid": "AAAAAAA", "score": 82 } )
    db.scores.insert( { "userid": "BBBBBBB", "score": 90 } )
其中：
+ **(1)查询：** 可以使用稀疏索引查询，返回完整集：**{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }**
+ **(2)查询：** 即使排序是通过索引字段进行的，MongoDB也不会选择稀疏索引来完成查询以返回完整的结果：  
    **{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }  
    { "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }  
    { "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }**
+ **(3)查询：** 使用hint()返回所需完整集：  
    **{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }  
    { "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }**

## 4. 其他事项

### 4.1 索引策略
**索引策略：**
+ **应用程序的最佳索引必须考虑许多因素，包括期望查询的类型，读取与写入的比率以及系统上的可用内存量。**
+ **在开发索引策略时，您应该深入了解应用程序的查询。在构建索引之前，映射将要运行的查询类型，以便您可以构建引用这些字段的索引。索引具有性能成本，但是对于大型数据集上的频繁查询而言，它们的价值更高。考虑应用程序中每个查询的相对频率以及查询是否证明索引是合理的。**
+ **设计索引的最佳总体策略是使用与您将在生产中运行的数据集类似的数据集来分析各种索引配置，以查看哪些配置性能最佳。检查为集合创建的当前索引，以确保它们支持您当前和计划的查询。如果不再使用索引，请删除索引。**
+ **通常，MongoDB仅使用一个索引来完成大多数查询。但是，$或查询的每个子句可能使用不同的索引，从2.6开始，MongoDB可以使用多个索引的交集。**

### 4.2 后续
&emsp;&emsp;后续还会有MongonDB索引优化，副本集以及分片总结、最重要还会总结在使用MongoDB实战以及实战过程出现的一些坑，可关注后续更新MongoDB系列。

**最后可关注公众号，一起学习,每天会分享干货，还有学习视频干货领取！**

![](https://user-gold-cdn.xitu.io/2020/4/14/171792690b19e0b8?w=350&h=129&f=png&s=17519)


