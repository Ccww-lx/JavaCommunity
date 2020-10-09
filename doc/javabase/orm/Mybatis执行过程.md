### 前言

在了解了MyBatis初始化加载过程后，我们也应该研究看看SQL执行过程是怎样执行？这样我们对于Mybatis的整个执行流程都熟悉了，在开发遇到问题也可以很快定位到问题。

更重要的，在面试中遇到面试官咨询Mybatis的知识点的时候，可以很顺畅的把这一套流程讲出来，面试官也会觉得你已掌握Mybatis知识点了，可能就不问了。赶紧瞄瞄

### 简介SQL执行过程

经过MyBatis初始化加载Sql执行过程所需的信息后，我们就可以通过 `SqlSessionFactory` 对象得到 `SqlSession` ,然后执行 SQL 语句了,接下来看看Sql执行具体过程，SQL大致执行流程图如下所示：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/431c3d6c877e4ebca565447189125aea~tplv-k3u1fbpfcp-zoom-1.image)

接下来我们来看看每个执行链路中的具体执行过程，

### SqlSession

SqlSession 是 MyBatis 暴露给外部使用的统一接口层，通过 `SqlSessionFactory` 创建，且其是包含和数据库打交道所有操作接口。

下面通过时序图描述 SqlSession 对象的创建流程：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fe09ac94b2f4489ac5e2b743052a275~tplv-k3u1fbpfcp-zoom-1.image)

在生成`SqlSession`的同时，基于`executorType`初始化好`Executor` 实现类。

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
```

最顶层的SqlSession接口已生成，那我们可以来看看sql的执行过程下一步是怎样的呢？怎样使用代理类`MapperProxy`。

### MapperProxy

 `MapperProxy` 是 `Mapper`接口与SQL 语句映射的关键，通过 `MapperProxy` 可以让对应的 SQL 语句跟接口进行绑定的，具体流程如下：

+ `MapperProxy`代理类生成流程
+ `MapperProxy`代理类执行操作

`MapperProxy`代理类生成流程

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bae83f3ac43e42a98088e3d3c7291e03~tplv-k3u1fbpfcp-zoom-1.image)

其中，`MapperRegistry` 是 `Configuration` 的一个属性，在解析配置时候会在`MapperRegistry` 中缓存了 `MapperProxyFactory` 的 `knownMappers` 变量`Map` 集合。

``MapperRegistry` 会根据`mapper`接口类型获取已缓存的`MapperProxyFactory`，`MapperProxyFactory`会基于`SqlSession`来生成`MapperProxy`代理对象，

```java
 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory)this.knownMappers.get(type);
        if (mapperProxyFactory == null) {
            throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
        } else {
            try {
                return mapperProxyFactory.newInstance(sqlSession);
            } catch (Exception var5) {
                throw new BindingException("Error getting mapper instance. Cause: " + var5, var5);
            }
        }
    }
```

当调用`SqlSession`接口时，`MapperProxy`怎么是实现的呢？`MyBatis` 的 `Mapper`接口 是通过动态代理实现的，调用 `Mapper` 接口的任何方法都会执行 `MapperProxy::invoke()` 方法，

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93991f72bfc44d7f8af8dd23dcf17f3c~tplv-k3u1fbpfcp-zoom-1.image)

```java
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            //Object类型执行
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);
            }
			//接口默认方法执行
            if (method.isDefault()) {
                if (privateLookupInMethod == null) {
                    return this.invokeDefaultMethodJava8(proxy, method, args);
                }

                return this.invokeDefaultMethodJava9(proxy, method, args);
            }
        } catch (Throwable var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
        MapperMethod mapperMethod = this.cachedMapperMethod(method);
        return mapperMethod.execute(this.sqlSession, args);
    }
```

但最终会调用到`mapperMethod::execute()` 方法执行，主要是判断是 `INSERT`、`UPDATE`、`DELETE` 、`SELECT` 语句去操作，其中如果是查询的话，还会判断返回值的类型。

```java
 public Object execute(SqlSession sqlSession, Object[] args) {
        Object result;
        Object param;
        switch(this.command.getType()) {
        case INSERT:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.insert(this.command.getName(), param));
            break;
        case UPDATE:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.update(this.command.getName(), param));
            break;
        case DELETE:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.delete(this.command.getName(), param));
            break;
        case SELECT:
            if (this.method.returnsVoid() && this.method.hasResultHandler()) {
                this.executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (this.method.returnsMany()) {
                result = this.executeForMany(sqlSession, args);
            } else if (this.method.returnsMap()) {
                result = this.executeForMap(sqlSession, args);
            } else if (this.method.returnsCursor()) {
                result = this.executeForCursor(sqlSession, args);
            } else {
                param = this.method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(this.command.getName(), param);
                if (this.method.returnsOptional() && (result == null || !this.method.getReturnType().equals(result.getClass()))) {
                    result = Optional.ofNullable(result);
                }
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + this.command.getName());
        }

        if (result == null && this.method.getReturnType().isPrimitive() && !this.method.returnsVoid()) {
            throw new BindingException("Mapper method '" + this.command.getName() + " attempted to return null from a method with a primitive return type (" + this.method.getReturnType() + ").");
        } else {
            return result;
        }
    }

```

通过以上的分析，总结出

- `Mapper`接口实际对象为代理对象` MapperProxy`；
- `MapperProxy `继承`InvocationHandler`，实现` invoke `方法；
- `MapperProxyFactory::newInstance()` 方法，基于 JDK 动态代理的方式创建了一个 `MapperProxy` 的代理类；
- 最终会调用到`mapperMethod::execute()` 方法执行，完成操作。
- 而且更重要一点是，`MyBatis` 使用的动态代理和普遍动态代理有点区别，没有实现类，只有接口，MyBatis 动态代理类图结构如下所示：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71dd617290b54bdba36907e15f127bfb~tplv-k3u1fbpfcp-zoom-1.image)

已以`SELECT` 为例， 调用会`SqlSession ::selectOne()` 方法。继续往下执行，会执行 `Executor::query()` 方法。

```java
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        List var5;
        try {
            MappedStatement ms = this.configuration.getMappedStatement(statement);
            var5 = this.executor.query(ms, this.wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
        } catch (Exception var9) {
            throw ExceptionFactory.wrapException("Error querying database.  Cause: " + var9, var9);
        } finally {
            ErrorContext.instance().reset();
        }

        return var5;
    }
```

执行到`Executor`类，那么我们来看看其究竟有什么？

### Executor

 `Executor`对象为SQL 的执行引擎，负责增删改查的具体操作，顶层接口`SqlSession`中都会有一个 `Executor` 对象，可以理解为 JDBC 中 `Statement` 的封装版。

`Executor` 是最顶层的是执行器，它有两个实现类，分别是`BaseExecutor`和 `CachingExecutor` 

+ `BaseExecutor` 是一个抽象类，实现了大部分 `Executor` 接口定义的功能，降低了接口实现的难度。`BaseExecutor `基于适配器设计模式之接口适配会有三个子类，分别是 `SimpleExecutor`、`ReuseExecutor` 和 `BatchExecutor`。

  + `SimpleExecutor` : 是 MyBatis 中**默认**简单执行器，每执行一次`update`或`select`，就开启一个`Statement`对象，用完立刻关闭`Statement`对象 

  + `ReuseExecutor` : 可重用执行器， 执行`update`或`select`，以sql作为`key`查找`Statement`对象，存在就使用，不存在就创建，用完后，不关闭`Statement`对象，而是放置于`Map<String, Statement>`内，供下一次使用。简言之，就是重复使用`Statement`对象 

  + `BatchExecutor` : 批处理执行器，用于执行update（没有select，JDBC批处理不支持select将多个 SQL 一次性输出到数据库, 

+ `CachingExecutor`: 缓存执行器，为`Executor`对象增加了**二级缓存**的相关功：先从缓存中查询结果，如果存在就返回之前的结果；如果不存在，再委托给`Executor delegate` 去数据库中取，`delegate` 可以是上面任何一个执行器。

在`Mybatis`配置文件中，可以指定默认的`ExecutorType`执行器类型，也可以手动给`DefaultSqlSessionFactory`的创建`SqlSession`的方法传递`ExecutorType`类型参数。 

看完`Exector`简介之后，继续跟踪执行流程链路分析，`SqlSession` 中的 `JDBC` 操作部分最终都会委派给 `Exector` 实现，`Executor::query()`方法,看看在`Exector`的执行是怎样的？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7950aecdb3054f33a2581f60a71560a4~tplv-k3u1fbpfcp-zoom-1.image)

每次查询都会先经过`CachingExecutor`缓存执行器， 会先判断二级缓存中是否存在查询 SQL ，如果存在直接从二级缓存中获取，不存在即为第一次执行，会直接执行SQL 语句，并创建缓存，都是由`CachingExecutor::query()`操作完成的。

```java
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
        BoundSql boundSql = ms.getBoundSql(parameterObject);
        CacheKey key = this.createCacheKey(ms, parameterObject, rowBounds, boundSql);
        return this.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }

public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
        //获取查询语句对应的二级缓存
    	Cache cache = ms.getCache();
        //sql查询是否存在在二级缓存中
   	    if (cache != null) {
            //根据 <select> 节点的配置，判断否需要清空二级缓存
            this.flushCacheIfRequired(ms);
            if (ms.isUseCache() && resultHandler == null) {
                this.ensureNoOutParams(ms, boundSql);
                //查询二级缓存
                List<E> list = (List)this.tcm.getObject(cache, key);
                if (list == null) {
                    //二级缓存没用相应的结果对象，调用封装的Executor对象的 query() 方法
                    list = this.delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                    //将查询结果保存到二级缓存中
                    this.tcm.putObject(cache, key, list);
                }

                return list;
            }
        }
		//没有启动二级缓存，直接调用底层 Executor 执行数据数据库查询操作
        return this.delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
```

如果在经过`CachingExecutor`缓存执行器（二级缓存）没有返回值的话，就会执行`BaseExecutor` 以及其的实现类，默认为`SimpleExecutor` ，首先会在一级缓存中获取查询结果，获得不到，最终会通过`SimpleExecutor::	()`去数据库中查询。

```java
 public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
        ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
        if (this.closed) {
            throw new ExecutorException("Executor was closed.");
        } else {
            //是否清除本地缓存
            if (this.queryStack == 0 && ms.isFlushCacheRequired()) {
                this.clearLocalCache();
            }

            List list;
            try {
                ++this.queryStack;
                //从一级缓存中，获取查询结果
                list = resultHandler == null ? (List)this.localCache.getObject(key) : null;
                //获取到结果，则进行处理
                if (list != null) {
                    this.handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
                } else {
                    //获得不到，则从数据库中查询
                    list = this.queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
                }
            } finally {
                --this.queryStack;
            }

            if (this.queryStack == 0) {
                //执行延迟加载
                Iterator var8 = this.deferredLoads.iterator();

                while(var8.hasNext()) {
                    BaseExecutor.DeferredLoad deferredLoad = (BaseExecutor.DeferredLoad)var8.next();
                    deferredLoad.load();
                }

                this.deferredLoads.clear();
                if (this.configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
                    this.clearLocalCache();
                }
            }

            return list;
        }
    }
```

那么`SimpleExecutor::doQuery()`如何去数据库中查询获取到结果呢？其实执行到这边mybatis的执行过程就从 `Executor`转交给  `StatementHandler`处理，

```java
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        Statement stmt = null;

        List var9;
        try {
            Configuration configuration = ms.getConfiguration();
            StatementHandler handler = configuration.newStatementHandler(this.wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
            stmt = this.prepareStatement(handler, ms.getStatementLog());
            var9 = handler.query(stmt, resultHandler);
        } finally {
            this.closeStatement(stmt);
        }

        return var9;
    }
```

 这样我们的执行链路分析已到`StatementHandler`了，现在让我们去一探究竟其原理



### StatementHandler

`StatementHandler`负责处理`Mybatis`与JDBC之间`Statement`的交互，即`Statement `对象与数据库进行交互，其为顶级接口,有4个实现类，其中三个是`Statement `对象与数据库进行交互类， 另外一个是路由功能的，

+ `RoutingStatementHandler`:  对 `Statement` 对象没有实际操作，主要负责另外三个`StatementHandler`的创建及调用， 而且在`MyBatis`执行时,使用的`StatementHandler `接口对象实际上就是 `RoutingStatementHandler` 对象。 
+ `SimpleStatementHandler`: 管理 Statement 对象， 用于简单SQL的处理 。
+ `PreparedStatementHandler`: 管理 Statement 对象，预处理SQL的接口 。
+ `CallableStatementHandler`：管理 Statement 对象，用于执行存储过程相关的接口 。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa3520be05804befa628c63fb24a446d~tplv-k3u1fbpfcp-zoom-1.image)

在经历过`Executor`后，基于初始化加载到`MapperState`中的`StatementType`的类型通`过Configuration.newStatementHandler()`方法中的`RoutingStatementHandler` 生成`StatementHandler`实际处理类。

```java
 public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        switch(ms.getStatementType()) {
        case STATEMENT:
            this.delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case PREPARED:
            this.delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case CALLABLE:
            this.delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        default:
            throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
        }

    }
```

现在先以`PreparedStatementHandler`预处理为例，接着Sql的执行链路来分析，`StatementHandler::query()`到`StatementHandler::execute()`真正执行Sql查询操作。

```java
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    String sql = boundSql.getSql();
    statement.execute(sql);
    return resultSetHandler.handleResultSets(statement);
  }
```

但执行真正查询操作之前，还进行哪些处理呢？还会进行`ParameterHandler`对 SQL 参数的预处理：对参数进行动态Sql映射，那么`ParameterHandler`又如何实现对参数进行动态映射的呢？

### ParameterHandler

`ParameterHandler` 参数处理器， 用来设置参数规则的，负责为sql 语句参数动态赋值，其有两个接口

- getParameterObject： 用于读取参数
- setParameters: 用于对 PreparedStatement 的参数赋值

当`SimpleExecutor`执行构造`PreparedStatementHandler`完，会调用`parameterize()`方法将`PreparedStatement`对象里SQL转交`ParameterHandler`实现类 `DefaultParameterHandler::setParameters()`方法 设置 `PreparedStatement` 的占位符参数 。

```java
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout());
   	//参数动态赋值
    handler.parameterize(stmt);
    return stmt;
  }
```

`DefaultParameterHandler::setParameters()`如何对SQL进行动态赋值呢？在执行前将已装载好的BoundSql对象信息进行使用 

```java
 public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
     //获取待动态赋值参数列表的封装parameterMappings
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
          //是否为输入参数
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
          //获取待动态参数属性名
          String propertyName = parameterMapping.getProperty();
          if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } else {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          //// 在通过 SqlSource 的parse 方法得到parameterMappings 的具体实现中，我们会得到parameterMappings 的 typeHandler
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          //获取jdbc数据类型
          JdbcType jdbcType = parameterMapping.getJdbcType();
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
            //
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
          } catch (TypeException | SQLException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          }
        }
      }
    }
  }
```

 执行完SQL 参数的预处理，当`StatementHandler::execute()`真正执行查询操作执行完后，有返回结果，需要对返回结果进行`ResultSetHandler`处理，现在看看最后的结果的处理流程。

### ResultSetHandler

`ResultSetHandler `结果解析器，将查询结果的`ResultSet` 转换成映射的对应结果(`java DTO`等)，其有三接口

+ `handleResultSets()`:处理结果集
+ `handleCursorResultSets()`：批量处理结果集
+ `handleOutputParameters()`：处理存储过程返回的结果集

其默认的实现为`DefaultResultSetHandler`,主要功能为：

+   处理`Statement` 执行后产生的结果集生成相对的输出结果、
+  处理存储过程执行后的输出参数  

那看看`DefaultResultSetHandler::handleResultSets()`如何处理？

+ 当有多个`ResultSet`的结果集合，每个ResultSet对应一个Object 对象，如果不考虑存储过程，普通的查询只有一个ResultSet
+  `ResultSetWrapper`封装了`ResultSet`结果集,其属性包含`ResultSet` ,`ResultMap`等 

```java
@Override
public List<Object> handleResultSets(Statement stmt) throws SQLException {
    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

    //当有多个ResultSet的结果集合，每个ResultSet对应一个Object 对象
    final List<Object> multipleResults = new ArrayList<>();

    int resultSetCount = 0;
    //获得首个 ResultSet 对象，并封装成 ResultSetWrapper 对象
    ResultSetWrapper rsw = getFirstResultSet(stmt);

    //获得 ResultMap 数组
    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    int resultMapCount = resultMaps.size();
    validateResultMapsCount(rsw, resultMapCount); // <3.1> 校验
    while (rsw != null && resultMapCount > resultSetCount) {
        //获得 ResultMap 对象
        ResultMap resultMap = resultMaps.get(resultSetCount);
        //处理 ResultSet ，将结果添加到 multipleResults 中
        handleResultSet(rsw, resultMap, multipleResults, null);
        //获得下一个 ResultSet 对象，并封装成 ResultSetWrapper 对象
        rsw = getNextResultSet(stmt);
        //清理
        cleanUpAfterHandlingResultSet();
        // resultSetCount ++
        resultSetCount++;
    }

    String[] resultSets = mappedStatement.getResultSets();
    if (resultSets != null) {
        while (rsw != null && resultSetCount < resultSets.length) {
            ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
            if (parentMapping != null) {
                String nestedResultMapId = parentMapping.getNestedResultMapId();
                ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                handleResultSet(rsw, resultMap, null, parentMapping);
            }
            rsw = getNextResultSet(stmt);
            cleanUpAfterHandlingResultSet();
            resultSetCount++;
        }
    }

    //如果是 multipleResults 单元素，则取首元素返回
    return ollapseSingleResultList(multipleResults);
}
```

其实在`ResultSetHandler`结果集处理是比较复杂的，这里只是简单的介绍一下，有兴趣的可以再深入研究一下，后期有空也会写。

执行到这边，Mybatis SQL执行基本完了，会把转换后的结果集返回到操作者。

### 结论

在SQL执行过程主要涉及了`SqlSession`，`MapperProxy`,`Executor`,`StatementHandler`,`ParameterHandler`以及`ResultSetHandler`，包括参数动态绑定，Sql执行查询数据库数据，结果返回集映射等，而且每个环节涉及的内容都很多，每个接口都可以抽出单独分析，后续有时间再一一详细的看看。后面还是再分析一下插件的应用。


