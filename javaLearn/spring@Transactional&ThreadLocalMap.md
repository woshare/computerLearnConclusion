
# Springboot Transactional事务 和ThreadLocalMap隐性传参

## 1,在ThreadLocalMap会存储很多信息，用于隐性传参
>1，其中包括org.springframework.jdbc.datasource.ConnectionHolder@72bdf389,>2org.apache.ibatis.session.defaults.DefaultSqlSessionFactory@49f41c2e=
org.mybatis.spring.SqlSessionHolder@32772777

## 2，org.springframework.jdbc.datasource.ConnectionHolder@72bdf389
```
={{
	CreateTime:"2021-05-11 14:46:05",
	ActiveCount:2,
	PoolingCount:7,
	CreateCount:9,
	DestroyCount:0,
	CloseCount:20280,
	ConnectCount:20282,
	Connections:[
		{ID:1017290886, ConnectTime:"2021-05-11 14:47:20", UseCount:9, LastActiveTime:"2021-05-11 14:59:12"", CachedStatementCount:8},
		{ID:352736418, ConnectTime:"2021-05-11 14:59:12", UseCount:1, LastActiveTime:"2021-05-11 14:59:53"", CachedStatementCount:1},
		{ID:1538445033, ConnectTime:"2021-05-11 14:46:08", UseCount:10, LastActiveTime:"2021-05-11 14:59:53"", CachedStatementCount:5},
		{ID:1862003061, ConnectTime:"2021-05-11 14:46:08", UseCount:56, LastActiveTime:"2021-05-11 14:59:53", LastKeepTimeMillis:"2021-05-11 14:59:08"", CachedStatementCount:11},
		{ID:1918746557, ConnectTime:"2021-05-11 14:46:08", UseCount:3, LastActiveTime:"2021-05-11 14:59:53", LastKeepTimeMillis:"2021-05-11 14:59:12"", CachedStatementCount:2},
		{ID:597508344, ConnectTime:"2021-05-11 14:46:08", UseCount:9858, LastActiveTime:"2021-05-11 14:59:53"", CachedStatementCount:12},
		{ID:1781740187, ConnectTime:"2021-05-11 14:46:17", UseCount:119, LastActiveTime:"2021-05-11 14:59:57", LastKeepTimeMillis:"2021-05-11 14:59:12"", CachedStatementCount:9},
		{ID:355178881, ConnectTime:"2021-05-11 14:46:17", UseCount:173, LastActiveTime:"2021-05-11 14:59:57"", CachedStatementCount:10}
	]
}

[
	{
	ID:1017290886, 
	poolStatements:[
		{hitCount:0,sql:"select count(*) from yy where id=? and  show_status=? and uid =?"	}
		]
	},
	{
	ID:352736418, 
	poolStatements:[
		{hitCount:0,sql:"select * from dailyresource_switcher order by id desc limit 1"	}
		]
	},
	{
	ID:1538445033, 
	poolStatements:[
		{hitCount:0,sql:"select count(*) from yy where id=? and  show_status=? and uid =?"	}
		]
	},
	{
	ID:1862003061, 
	poolStatements:[
		{hitCount:0,sql:"select count(*) from yy where id=? and  show_status=? and uid =?"	}
		]
	},
	{
	ID:1918746557, 
	poolStatements:[
	{hitCount:4124,sql:"select count(*) from cc where id=? and  show_status=? and uid =?"	}]
	},
	{
	ID:597508344, 
	poolStatements:[
		{hitCount:4124,sql:"select count(*) from cc where id=? and  show_status=? and uid =?"	}
		]
	},
	{
	ID:1781740187, 
	poolStatements:[
		{hitCount:0,sql:"select count(*) from yy where id=? and  show_status=? and uid =?"	}
		]
	},
	{
	ID:355178881, 
	poolStatements:[

		{hitCount:20,sql:"select count(*) from xx where id=? and  show_status=? and uid =?"	}
		]
	}
]
```
## 3,debug测试场景
>1,service1 { this-sql1-insert；this-sql2-update；service2-sql3-select；} ，其中事务：PROPAGATION_REQUIRED,ISOLATION_DEFAULT，工作线程：http-nio-8080-exec-1


## mapper.sqlMethod 会动态代理，调用SqlSessionTemplate 中的invoke ，SqlSession sqlSession = SqlSessionUtils.getSqlSession 则是通过NameThreadLocal中获取sqlSession，同一个事务中sqlSession，以及org.mybatis.spring.SqlSessionHolder 是同一个，就算是service1中调用了service2，其中，org.apache.ibatis.session.defaults.DefaultSqlSessionFactory应该是全局唯一的
```
public class SqlSessionTemplate implements SqlSession, DisposableBean {
  private final SqlSessionFactory sqlSessionFactory;
  private final ExecutorType executorType;
  private final SqlSession sqlSessionProxy;
  private final PersistenceExceptionTranslator exceptionTranslator;

private class SqlSessionInterceptor implements InvocationHandler {
    private SqlSessionInterceptor() {
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      SqlSession sqlSession = SqlSessionUtils.getSqlSession(SqlSessionTemplate.this.sqlSessionFactory, SqlSessionTemplate.this.executorType, SqlSessionTemplate.this.exceptionTranslator);

      Object unwrapped;
      try {
        Object result = method.invoke(sqlSession, args);
        if (!SqlSessionUtils.isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
          sqlSession.commit(true);
        }

        unwrapped = result;
      } catch (Throwable var11) {
        unwrapped = ExceptionUtil.unwrapThrowable(var11);
        if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
          SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
          sqlSession = null;
          Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException)unwrapped);
          if (translated != null) {
            unwrapped = translated;
          }
        }

        throw (Throwable)unwrapped;
      } finally {
        if (sqlSession != null) {
          SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);//
        }

      }

      return unwrapped;
    }
  }

}

public static void closeSqlSession(SqlSession session, SqlSessionFactory sessionFactory) {
    Assert.notNull(session, "No SqlSession specified");
    Assert.notNull(sessionFactory, "No SqlSessionFactory specified");
    SqlSessionHolder holder = (SqlSessionHolder)TransactionSynchronizationManager.getResource(sessionFactory);
    if (holder != null && holder.getSqlSession() == session) {
      LOGGER.debug(() -> {
        return "Releasing transactional SqlSession [" + session + "]";
      });
      holder.released();  //引用计数-1
    } else {
      LOGGER.debug(() -> {
        return "Closing non transactional SqlSession [" + session + "]";
      });
      session.close();
    }

  }
```

## SqlSessionHolder 会注册保存到NameThreadLocal，通过sqlSessionHolder来执行sqlSession

```
private static void registerSessionHolder(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator, SqlSession session) {
    if (TransactionSynchronizationManager.isSynchronizationActive()) {
      Environment environment = sessionFactory.getConfiguration().getEnvironment();
      if (environment.getTransactionFactory() instanceof SpringManagedTransactionFactory) {
        LOGGER.debug(() -> {
          return "Registering transaction synchronization for SqlSession [" + session + "]";
        });
        SqlSessionHolder holder = new SqlSessionHolder(session, executorType, exceptionTranslator);
        TransactionSynchronizationManager.bindResource(sessionFactory, holder);
        TransactionSynchronizationManager.registerSynchronization(new SqlSessionUtils.SqlSessionSynchronization(holder, sessionFactory));
        holder.setSynchronizedWithTransaction(true);
        holder.requested();
      } else {
        if (TransactionSynchronizationManager.getResource(environment.getDataSource()) != null) {
          throw new TransientDataAccessResourceException("SqlSessionFactory must be using a SpringManagedTransactionFactory in order to use Spring transaction synchronization");
        }

        LOGGER.debug(() -> {
          return "SqlSession [" + session + "] was not registered for synchronization because DataSource is not transactional";
        });
      }
    } else {
      LOGGER.debug(() -> {
        return "SqlSession [" + session + "] was not registered for synchronization because synchronization is not active";
      });
    }

  }
```

## TransactionSynchronizationManager 下会注册这些sqlSession或sqlSessionHolder到ThreadLocal中
```
private static final ThreadLocal<Map<Object, Object>> resources =
			new NamedThreadLocal<>("Transactional resources");

	private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
			new NamedThreadLocal<>("Transaction synchronizations");

	private static final ThreadLocal<String> currentTransactionName =
			new NamedThreadLocal<>("Current transaction name");

	private static final ThreadLocal<Boolean> currentTransactionReadOnly =
			new NamedThreadLocal<>("Current transaction read-only status");

	private static final ThreadLocal<Integer> currentTransactionIsolationLevel =
			new NamedThreadLocal<>("Current transaction isolation level");

	private static final ThreadLocal<Boolean> actualTransactionActive =
			new NamedThreadLocal<>("Actual transaction active");
```


## 4，结论：
>1，工作线程通过threadLocalMap保存了很多信息，其中包括事务信息，这样service1调用service2，隐性传递事务信息
>2，service1调用service2，使用的同一个sqlSession，例如：org.apache.ibatis.session.defaults.DefaultSqlSessionFactory@49f41c2e=org.mybatis.spring.SqlSessionHolder@32772777
>3，NameThreadLocal<sqlSessionFactory,SqlSessionHolder>-->sqlSessionHolder-->sqlSession->Excutor

>Spring的自定义事务的一些机制，其中当前线程事务管理器是整个事务的核心与中轴，当前有事务时，会初始化当前线程事务管理器的synchronizations，即激活了当前线程同步管理器，当Mybatis访问数据库会首先从当前线程事务管理器获取SqlSession，如果不存在就会创建一个会话，接着注册会话到当前线程事务管理器中，如果当前有事务，则会话不关闭也不commit，Mybatis还自定义了一个TransactionSynchronization，用于事务每次状态发生时回调处理

>Mybatis自己也实现了一个自定义的事务同步回调器SqlSessionSynchronization，在注册SqlSession的同时，也会将SqlSessionSynchronization注册到当前线程事务管理器中，它的作用是根据事务的完成状态回调来处理线程资源，即当前如果有事务，那么当每次状态发生时就会回调事务同步器，具体细节可移步至Spring的org.springframework.transaction.support包
