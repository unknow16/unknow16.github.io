---
title: Mybatis拦截器实现源码解析
date: 2018-02-26 23:42:30
tags: Mybatis
---

### 拦截器调用流程

http://blog.csdn.net/moshenglv/article/details/52075341

1. 编写好拦截器，在sqlMapConfig配置文件中配置上，运行时会解析添加到Configuration中的InterceptorChain实例
* Configuration类中持有一个InterceptorChain实例，
```
protected final InterceptorChain interceptorChain = new InterceptorChain();
```

* InterceptorChain类
```
/**
 * @author Clinton Begin
 */
public class InterceptorChain {

    //采用List保存拦截器
  private final List<Interceptor> interceptors = new ArrayList<Interceptor>();

    //创建上面四种类型的*Handler时，调用该方法，
    //遍历执行拦截器的plugin方法，并传入目标对象
  public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
      target = interceptor.plugin(target);
    }
    return target;
  }

  public void addInterceptor(Interceptor interceptor) {
    interceptors.add(interceptor);
  }
  
  public List<Interceptor> getInterceptors() {
    return Collections.unmodifiableList(interceptors);
  }

}
```

2. 通过配置文件创建SqlSessionFactory(实现为DefaultSqlSessionFactory)

```
InputStream inputStream = Resources.getResourceAsStream("sqlMapConfig.xml");
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
* 通过sessionFactory创建sqlSession，同时然后创建Executor
```
//1. 客户端调用
SqlSession openSession = sessionFactory.openSession();

//2. DefaultSqlSessionFactory中的openSession()
public SqlSession openSession() {
return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
}

private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
        
        //Environment中包含数据源和事务管理器
      final Environment environment = configuration.getEnvironment();
        //创建事务
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        //创建执行器Executor,此时在newExecutor中会执行InterceptorChain的pluginAll(),详细见下文。
      final Executor executor = configuration.newExecutor(tx, execType);
        //返回sqlSesssion
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
    

```
* Configuration中的四个创建Executor、StatementHandler、ParameterHandler、ResultSetHandler的方法中调用了interceptorChain.pluginAll()方法，所以说只能拦截这四种类型的方法调用。

```
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    
    //拦截器被调用
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
  }

  public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    
    //拦截器被调用
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }

  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    //主要作用是分发，它根据配置Statement类型创建真正执行数据库操作的StatementHandler，并将其保存到delegate属性里。
    //跟StatementType中statementType判断，默认StatementType.PREPARED,见下文
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    
    //拦截器被调用
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
  }

  public Executor newExecutor(Transaction transaction) {
    return newExecutor(transaction, defaultExecutorType);
  }

  public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction); //默认该Executor
    }
    
    // 如果配置文件中配置<cache />标签，会创建CachingExecutor装饰SimpleExecutor
    // 其中持有一个SimpleExecutor的代理对象
    // 先从缓存中找，找不到调用SimpleExecutor执行数据查询
    if (cacheEnabled) { 
      executor = new CachingExecutor(executor);
    }
    
    //拦截器被调用
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
```


```
//创建RoutingStatementHandler
  public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {

    switch (ms.getStatementType()) {
      case STATEMENT:
        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
        
        //走这里，new PreparedStatementHandler时，创建了parameterHandler,resultSetHandler
      case PREPARED:
        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case CALLABLE:
        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      default:
        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
    }

  }
  
  //new PreparedStatementHandler时，创建了parameterHandler,resultSetHandler
  this.parameterHandler = configuration.newParameterHandler(mappedStatement, parameterObject, boundSql);
    this.resultSetHandler = configuration.newResultSetHandler(executor, mappedStatement, rowBounds, parameterHandler, resultHandler, boundSql);
  }
```

3. 真正执行sql操作时，才会创建StatementHandler、ParameterHandler、ResultSetHandler并调用pluginAll()
```

User user = openSession.selectOne("com.fuyi.mybatis_demo.mapper.UserMapper.findById", 3);
System.out.println(user);
```
* selectOne实际调用SimpleExecutor中的doQuery()

```
//SimpleExecutor中

  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      
      //创建StatementHandler，同时执行拦截器链,去上文看该方法，实际创建RoutingStatementHandler实例，再看上文RoutingStatementHandler构造方法
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      
      //创建PrepareStatement，其中才获取Connection，并设置sql参数
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }
  
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    //获取连接
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout());
    
    //设置参数，并根据配置文件中的java和jdbc类型转换
    //PreparedStatement中持有一个parameterHandler，它在创建StatementHandler时创建的。
    handler.parameterize(stmt);
    return stmt;
  }
  
//PreparedStatement中
  public void parameterize(Statement statement) throws SQLException {
    //调用StatementHandler设置参数
    parameterHandler.setParameters((PreparedStatement) statement);
  }
```

* StatementHandler的默认实现类是RoutingStatementHandler。
* RoutingStatementHandler的主要功能是分发，它根据配置Statement类型创建真正执行数据库操作的StatementHandler，并将其保存到delegate属性里。






