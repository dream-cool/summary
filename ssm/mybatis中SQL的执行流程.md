总体  获取sqlSession ->  获取mapper ->执行接口

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
UserDao userDao = sqlSession.getMapper(UserDao.class);
return userDao.queryById("1");
```



openSession()流程

这里mybatis默认将从DefaultSqlSessionFactory获取sqlSession，通过spring注入对应的mapper将使用SqlSessionTemplate获取sqlSession。

DefaultSqlSessionFactory.openSession()最终都将调用openSessionFromDataSource()

```java
    //获取环境变量
    final Environment environment = configuration.getEnvironment();
	//获取事物工厂
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
	//获取执行器
    final Executor executor = configuration.newExecutor(tx, execType);
	//获取默认连接
    return new DefaultSqlSession(configuration, executor, autoCommit);
```

获取执行器的过程将根据execType的不同生成BatchExecutor、ReuseExecutor、SimpleExecutor、CachingExecutor这里面的一种，但最后都将通过InterceptorChain.pluginAll将配置存在的plugin对executor进行包装。



常规的通过SqlSession.getMapper(UserDao.class)获取对应的dao。

```
MapperRegistry.getMapper()
MapperProxyFactory.newInstance()
Proxy.newProxyInstance()
//最终获取到对应的mapper对象，其实本质也就是  MapperProxy对象
```



流程如下

调用对应mapper的接口方法时，MapperProxy.invoke()，将对各接口进行增强处理。使得其将执行MapperMethod.execute()方法。根据MapperMethod.type确定其执行类型，最终sqlSession.selectOne(command.getName(), param);。

所以这根直接通过 sqlSession调用sqlSession.selectOne(“com.clt.dao.UserDao.queryById”)是一致的。而不管是调用selectOne还是selectList，mybatis都将只会执行selectList，如果是selectOne则再对结果集的数量进行判断返回第一条数据而已。

在selectList中，根据传来的statement，也就是MappedStatement的id，在Configuration拿到对应的MappedStatement。通过DefaultSqlSession.executor.query()执行查询。如果前面的获取执行器中，如果存在插件对执行器进行了包装(包装的过程本质也是动态代理增强)，则query方法将执行Plugin.invoke()，从而在相应的插件拦截器里面执行executor.query方法。执行器中，将试图从一级缓存中获取数据，如果没有则执行queryFromDatabase()。过程中，在Configuration.newStatementHandler生成StatementHandler，通过prepareStatement获取Statement并执行。 PreparedStatementLogger.invoke() 再到DruidPooledPreparedStatement.execute()