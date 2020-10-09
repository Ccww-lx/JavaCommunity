>该系列文章收录在公众号【Ccww技术博客】，原创技术文章早于博客推出

### 前言

`MyBatis`不管在是平时的使用还是在面试中都必须掌握一个知识点，`MyBatis` 是支持自定义 SQL、存储过程和高级映射的类持久框架，跟数据库打交道的一个开源持久化框架

我们来看看`MyBatis`架构：

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8a975d90bea4ab3ae5659a12fd09558~tplv-k3u1fbpfcp-zoom-1.image)

`MyBatis`的整体架构分为三层：

1. 基础支持层
2. 核心处理层
3. 接口层

我们这边主要讲解一下核心处理层组件

### 核心处理层

在核心处理层中，实现了 `MyBatis`的核心处理流程： `MyBatis`的**初始化**以及完成一次**数据库操作**的涉及的全部流程 ,主要模块分为：

- 配置解析
- SQL 解析与参数映射
- SQL 执行与结果集映射
- 插件

####  配置解析

配置解析对应模块: `builder` 和 `mapping` ，主要组件为:

- `XMLConfigBuilder`: 解析`mybatis-config.xml` 配置文件
- `XMLMapperBuilder`:解析映射配置文件`Mapper.xml`
- `XMLStatementBuilder`:主要负责解析 Statement 配置，即 `<select />`、`<insert />`、`<update />`、`<delete />` 标签
- `MapperAnnotationBuilder`:负责解析 Mapper 接口上的注解

在 Mybatis 初始化过程中，会加载 `mybatis-config.xml` 配置文件、加载映射配置文件以及 Mapper 接口中的注解`@Mapper`信息,

经`XML解析properties配置ConfigBuilder::parse`解析的配置信息会形成相应的对象并保存到 `Configration` 对象中。之后，根据基于`Configration` 创建SqlSessionFactory 对象,待 `Mybatis` 初始化完成后，可以通过 `SqlSessionFactory` 创建 `SqlSession` 对象并开始数据库操作。

`Configuration`: MyBatis 所有的配置信息都维持在 Configuration 对象之中。

#### SQL 解析与参数映射

SQL 解析模块： `scripting` ，`XMLLanguageDriver,XMLScriptBuilder`实现了动态 SQL 语句，其提供了多种动态 SQL语句对应的节点。比如：

- `<where>` 节点、
- `<set>` 节点、
- `<foreach>` 节点等 。

通过这些节点的组合使用， 几乎可以编写出所有满足需要的 SQL。

先解析映射文件中定义的动态 SQL 节点，然后可以根据用户传入的参数，将已解析的SQL 语句中的占位符，绑定用户传入的实参，形成数据库能执行的SQL 语句



#### SQL 执行与结果集映射

SQL 执行与结果集映射对应的模块： `executor`（执行器） 和 `cursor`（结果游标） 模块等。提供操作接口到数据处理后返回的一系列操作，主要模块有：

- `SqlSession`： MyBatis 核心 API，主要用来执行命令，获取映射，管理事务。接收开发人员提供 `Statement Id` 和参数，并返回操作结果。
- `Executor` ：执行器，是 `MyBatis` 调度的核心，负责 SQL 语句的生成以及查询缓存（一级/二级缓存）的维护，它会将数据库相关操作委托给 `StatementHandler`完成。
- `StatementHandler` : 封装了`JDBC Statement` 操作，负责对 `JDBC Statement` 的操作，如设置参数、将Statement 结果集转换成 List 集合。
- `ParameterHandler` : 负责对用户传递的参数转换成 `JDBC Statement` 所需要的参数。
- `ResultSetHandler` : 负责将 JDBC 返回的 `ResultSet` 结果集对象转换成 List 类型的集合。
- `TypeHandler` : 用于 Java 类型和 JDBC 类型之间的转换。
- `MappedStatement` : 动态 SQL 的封装
- `SqlSource` : 表示从 XML 文件或注释读取的映射语句的内容，它创建将从用户接收的输入参数传递给数据库的 SQL。

![整体过程](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6d150b34a1843b983a99008efd3877c~tplv-k3u1fbpfcp-zoom-1.image)

#### 插件层

插件层对应 模块：`plugin` 模块主要拦截器实现`Interceptor`的，用户可以通过自定义插件来改变 `Mybatis` 的默认行为。

虽然Mybatis 自身的功能非常强大，但并不能完美符合所有的应用场景，因此 `MyBatis` 提供了插件接口，我们可以通过添加用户自定义插件的方式对 `MyBatis` 进行扩展，例如，我们可以拦截 SQL 语句并对其进行重写。

但MyBatis只允许使用插件来拦截的这些方法调用：**`Executor`、`ParameterHandler`、`ResultSetHandler`、`StatementHandler`** 接口

> 由于用户自定义插件会影响 MyBatis 的核心行为，因此在使用自定义插件之前，开发人员需要了解 MyBatis 内部的原理，这样才能编写出安全、高效的插件。

### 接口层

接口层对应 `session` 模块，其主要包括:

- `SqlSessionFactory和SqlSession`是MyBatis核心接口，接口中有执行命令，获取映射，管理事务操作，其也是与上层应用交互的桥梁。接口层在接收到调用请求时，会调用核心处理层的相应模块来完成具体的数据库操作。
- `Configuration`: MyBatis 所有的配置信息都维持在 `Configuration` 对象之中

### 基础支持层

基础支持层，包含整个` MyBatis` 的基础模块，这些模块为核心处理层的功能提供了良好的支撑。

#### 反射模块

反射模块对应： `reflection` 模块，Mybatis 中的反射模块，对 Java 反射进行了很好的封装，提供了简易的 API，方便上层调用，并且对反射操作进行了一系列的优化，例如缓存了类的元数据，提高了反射操作的性能

#### 类型模块

类型转换模块对应： `type` 模块，`MyBatis` 为简化配置文件提供了**别名机制**，该机制是类型转换模块的主要功能之一。

类型转换模块的另一个功能是**实现 JDBC 类型与 Java 类型之间**的转换，该功能在为 SQL 语句绑定实参以及映射查询结果集时都会涉及：

- 在为 SQL 语句绑定实参时，会将数据由 Java 类型转换成 JDBC 类型。

- 在映射结果集时，会将数据由 JDBC 类型转换成 Java 类型

#### 日志模块

日志模块对应： `logging` 包，MyBatis 作为一个设计优良的框架，除了提供详细的日志输出信息，还要能够集成多种日志框架，其日志模块的一个主要功能就是**集成第三方日志框架**，方便开发人员和测试人员快速定位 Bug 代码

####  资源加载模块

对应 `io` 包，资源加载模块，主要是对类加载器进行封装，确定类加载器的使用顺序，并提供了加载类文件以及其他资源文件的功能 。

#### 解析器模块

解析器模块对应： `parsing` 包，解析器模块，主要提供了两个功能:

- 一个功能，是对`XPath` 进行封装，为 MyBatis 初始化时解析 `mybatis-config.xml` 配置文件以及映射配置文件提供支持。

- 另一个功能，是为处理动态 SQL 语句中的占位符提供支持

#### 数据源模块

数据源模块对应： `datasource` 包，`MyBatis` 自身提供了相应的数据源实现，当然 MyBatis 也提供了与第三方数据源集成的接口，这些功能都位于数据源模块之中。

数据源是实际开发中常用的组件之一。现在开源的数据源都提供了比较丰富的功能，例如，连接池功能、检测连接状态等，选择性能优秀的数据源组件对于提升 ORM 框架乃至整个应用的性能都是非常重要的。

#### 事务模块

事务模块对应： `transaction` 包，MyBatis 对数据库中的事务进行了抽象，其自身提供了**相应的事务接口和简单实现**。

在很多场景中，`MyBatis` 会与 `Spring` 框架集成，并由 **Spring 框架管理事务**。

#### 缓存模块

缓存摸对应： `cache` 包，`MyBatis` 中提供了**一级缓存和二级缓存**，其都是依赖于基础支持层中的缓存模块实现的。

而且在优化系统性能时，优化数据库性能是一个比较可行的，而增加缓存则是优化数据库时最有效的手段之一，正确、合理地使用缓存可以将一部分数据库请求拦截在缓存这一层。

这里需要注意的是，由于MyBatis 自带的缓存与MyBatis以及整个应用是运行在同一个 JVM 中的，共享同一块堆内存。如果这两级缓存中的数据量较大， 则可能影响系统中其他功能的运行，所以当需要缓存大量数据时，优先考虑使用 Redis、Memcache 等缓存产品

#### Binding 模块

对应 `binding` 包，MyBatis 通过 `Binding` 模块，将用户自定义的 `Mapper` 接口与映射配置文件联系起来，系统可以通过调用自定义 Mapper 接口中的方法执行相应的 SQL 语句完成数据库操作，并且会在运行期间进行校验映射文件是否出现语法拼写错误，可以尽早避免这种错误，提供程序的可用性。

> 值得注意的是，开发人员无须编写自定义 `Mapper` 接口的实现，MyBatis 会自动为其创建动态代理对象。在有些场景中，自定义 `Mapper` 接口可以完全代替映射配置文件，但有的映射规则和 SQL 语句的定义还是写在映射配置文件中比较方便，例如动态 SQL 语句的定义。

#### 注解模块

对应 `annotations` 包，`MyBatis` 提供了**注解**的方式，使得我们方便的在 `Mapper` 接口上编写简单的数据库 SQL 操作代码，而无需像之前一样，必须编写 SQL 在 XML 格式的 Mapper 文件中。

#### 异常模块

对应 `exceptions` 包。定义了 MyBatis 专有的` PersistenceException `和 `TooManyResultsException` 异常。

##  总结

经过以上模块大概了解`Mybatis`后，这样可以更好为我们后续研读`MyBatis`源码有着很大帮助，后续将研读一下SQL 执行的流程，提供Mybatis自动化能力。在最后我们在来看看模块的架构图：
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db006050c27d4f7492c04ced2f5a0a41~tplv-k3u1fbpfcp-zoom-1.image)

