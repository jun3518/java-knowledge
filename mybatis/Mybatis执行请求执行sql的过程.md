## 接口层

### SqlSession

SqlSession是Mybatis核心接口之一，也是Mybatis接口层的主要组成部分，对外提供Mybatis常用API。Mybatis提供了两个SqlSession接口的实现DefaultSqlSession和SqlSessionManager，默认是的DefaultSqlSession。

```java
public interface SqlSession extends Closeable {
    // 泛型方法，参数表示使用的查询SQL语句，返回值为查询的结果对象
    <T> T selectOne(String statement);
    // 第二个参数表示需要用户传入的实参，也就是SQL语句绑定的实参
    <T> T selectOne(String statement, Object parameter);
    <E> List<E> selectList(String statement);
    <E> List<E> selectList(String statement, Object parameter);
    <K, V> Map<K, V> selectMap(String statement, String mapKey);
    <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey);
    <T> Cursor<T> selectCursor(String statement);
    <T> Cursor<T> selectCursor(String statement, Object parameter);
    void select(String statement, Object parameter, ResultHandler handler);
    void select(String statement, ResultHandler handler);
    int insert(String statement);
    int insert(String statement, Object parameter);
    int update(String statement);
    int update(String statement, Object parameter);
    int delete(String statement);
    int delete(String statement, Object parameter);
    void commit();
    void commit(boolean force);
    void rollback();
    void rollback(boolean force);
    List<BatchResult> flushStatements();
    void clearCache();
    Configuration getConfiguration();
    <T> T getMapper(Class<T> type);
    Connection getConnection();
}
```

对于DefaultSqlSession中核心字段的含义：

```java
public class DefaultSqlSession implements SqlSession {
    // 配置对象
    private final Configuration configuration;
    // 底层依赖的Execuctor对象，所有数据库相关的操作全部封装到Executor接口实现中
    private final Executor executor;
    // 是否自动提交事务
    private final boolean autoCommit;
    // 当前缓存中是否有脏数据
    private boolean dirty;
    // 为防止用户忘记关闭已打开的游标对象，会通过cursorList字段记录由该SqlSession对象生成的游标
    private List<Cursor<?>> cursorList;
}
```



### SqlSessionFactory

SqlSessionFactory负责创建SqlSession对象，其中只包含了多个openSession()方法的重载，可以通过其参数指定事务的隔离级别、底层使用Execucor的类型以及是否自动提交事务等方面的配置。

DefaultSqlSessionFactory是一个具体工厂类，实现了SqlSessionFactory接口。DefaultSqlSessionFactory主要提供了两种创建DefaultSqlSession对象的方式，一种是通过数据源获取数据库连接，并创建Executor对象以及DefaultSqlSession对象：

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, 
                                TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
        // 获取mybatis-config.xml配置文件中配置的Environment对象
        final Environment environment = configuration.getEnvironment();
        // 获取TransactionFactory对象
        final TransactionFactory transactionFactory = 
            getTransactionFactoryFromEnvironment(environment);
        // 创建Transaction对象
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, 
                                               autoCommit);
        // 根据配置创建Executor对象
        final Executor executor = configuration.newExecutor(tx, execType);
        return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
        // 关闭Transaction
        closeTransaction(tx); 
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

另一种方式是用户提供数据库连接对象，DefaultSqlSessionFactory会使用该数据库连接对象创建Executor对象以及DefaultSqlSession对象：

```java
private SqlSession openSessionFromConnection(ExecutorType execType, 
                                             Connection connection) {
    try {
        boolean autoCommit;
        try {
            // 获取当前连接的事务是否为自动提交方式
            autoCommit = connection.getAutoCommit();
        } catch (SQLException e) {
            // 当前数据库驱动提供的连接不支持事务，则可能会抛出异常
            autoCommit = true;
        }      
        // 获取mybatis-config.xml配置文件中配置的Environment对象
        final Environment environment = configuration.getEnvironment();
        // 获取得TransactionFactory对象
        final TransactionFactory transactionFactory = 
            getTransactionFactoryFromEnvironment(environment);
        // 创建Transaction对象
        final Transaction tx = transactionFactory.newTransaction(connection);
        // 根据配置创建Executor对象
        final Executor executor = configuration.newExecutor(tx, execType);
        // 创建DefaultSqlSession对象
        return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

### SqlSessionManager

SqlSessionManager同时实现了SqlSession接口和SqlSessionFactory接口，也就同时提供了SqlSessionFactory创建SqlSession对象以及SqlSession操纵数据库的功能。

SqlSessionManager中各个字段的含义如下：

```java
public class SqlSessionManager implements SqlSessionFactory, SqlSession {
	// 底层封装的SqlSessionFactory对象
    private final SqlSessionFactory sqlSessionFactory;
    // localSqlSession中记录的SqlSession对象的代理对象，在SqlSessionManager初始化时，会使用JDK
    // 动态代理的方式为localSqlSession创建代理对象
    private final SqlSession sqlSessionProxy;
	// ThreadLocal	变量，记录一个与当前线程绑定的SqlSession对象
    private final ThreadLocal<SqlSession> localSqlSession = 
        new ThreadLocal<SqlSession>();
}
```

SqlSessionManager与DefaultSqlSessionFactory的主要不同点是SqlSessionManager提供了两种模式：第一种模式与DefaultSqlSessionFactory的行为相同，同一线程每次通过SqlSessionManager对象访问数据库时，都会创建新的DefaultSqlSession对象完成数据库操作；第二种模式是SqlSessionManager通过localSqlSession这个ThreadLocal变量，记录与当前线程绑定的SqlSession对象，供当前线程循环使用，从而避免在同一线程多次创建SqlSession对象带来的性能损失。

如果要创建SqlSessionManager对象，需要调用其newInstance()方法。

```java
// 通过newInstance()方法创建SqlSessionManager对象
public static SqlSessionManager newInstance(SqlSessionFactory sqlSessionFactory) {
    return new SqlSessionManager(sqlSessionFactory);
}		
// SqlSessionManager的私有构造方法
private SqlSessionManager(SqlSessionFactory sqlSessionFactory) {
    this.sqlSessionFactory = sqlSessionFactory;
    // 使用动态代理的方式生成SqlSession的代理对象
    this.sqlSessionProxy = (SqlSession) Proxy.newProxyInstance(
        SqlSessionFactory.class.getClassLoader(),
        new Class[]{SqlSession.class},
        new SqlSessionInterceptor());
}
```

SqlSessionManager中实现的SqlSession接口方法，都是直接调用sqlSessionProxy属性字段记录的SqlSession代理对象的响应方法实现的。在创建该代理对象时使用的InvocationHandler对象是SqlSessionInterceptor对象，它是定义在SqlSessionManager中的内部类，其invoke()方法实现如下：

```java
private class SqlSessionInterceptor implements InvocationHandler {
    public SqlSessionInterceptor() {
        // Prevent Synthetic Access
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 获取当前线程绑定的SqlSession对象
        final SqlSession sqlSession = SqlSessionManager.this.localSqlSession.get();
        // 第二种模式
        if (sqlSession != null) {
            try {
                // 调用真正的SqlSession对象，完成数据库的相关操作
                return method.invoke(sqlSession, args);
            } catch (Throwable t) {
                throw ExceptionUtil.unwrapThrowable(t);
            }
        } else {
         	// 第一种模式
            // 如果当前线程未绑定SqlSession对象，则创建新的SqlSession对象
            final SqlSession autoSqlSession = openSession();
            try {
                // 通过新建的SqlSession对象完成数据库操作
                final Object result = method.invoke(autoSqlSession, args);
                // 提交事务
                autoSqlSession.commit();
                return result;
            } catch (Throwable t) {
                // 回滚事务
                autoSqlSession.rollback();
                throw ExceptionUtil.unwrapThrowable(t);
            } finally {
                // 关闭上面创建的SqlSession对象
                autoSqlSession.close();
            }
        }
    }
}
```

通过对SqlSessionInterceptor的分析可知，第一种模式中新建的SqlSession在使用完成后会立即关闭。在第二种模式中，与当前线程绑定的SqlSession对象需要先通过SqlSessionManager.startManagedSession()方法进行设置：

```java
public void startManagedSession() {
    this.localSqlSesssion.set(openSession());
}
```

当需要提交/回滚或是关闭localSqlSession中记录的SqlSession对象时，需要通过SqlSessionManager.commit()、rollback()以及close()方法完成，其中会先检测当前线程是否绑定了SqlSession对象，荣誉感未绑定，则抛异常；如果绑定了，则调用该SqlSession对象的响应方法。



## DefaultSqlSession#getMapper请求

DefaultSqlSession#getMapper是Mybatis请求的入口：

```java
// org.apache.ibatis.session.defaults.DefaultSqlSession#getMapper
public <T> T getMapper(Class<T> type) {
    return configuration.<T>getMapper(type, this);
}
// org.apache.ibatis.session.Configuration#getMapper
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}
// org.apache.ibatis.binding.MapperRegistry#getMapper
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = 
        (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
        throw new BindingException("...");
    }
    try {
        return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
        throw new BindingException("...");
    }
}
// MapperProxyFactory#newInstance(SqlSession)
public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, 
                                                          methodCache);
    return newInstance(mapperProxy);
}
//MapperProxyFactory#newInstance(MapperProxy<T>)
protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), 
                                      new Class[] { mapperInterface }, mapperProxy);
}
```

其中，type是接口类的Class对象。一个type类型的Class对象会在MapperRegistry#knownMappers属性中映射一个MapperProxyFactory对象。MapperProxyFactory#methodCache的Map属性中，type类型接口的方法会对应一个MapperMethod对象。

对于MapperProxy类，它实现了java.lang.reflect.InvocationHandler接口，Mybatis就是通过MapperProxy类，实现对接口和Mapper的关联，操作接口的某个方法时，就是操作MapperMethod对应的SQL节点。

当执行type.method时，实际上就是执行MapperProxy#invoke方法：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // method是Object中声明的方法时，直接放行。
    if (Object.class.equals(method.getDeclaringClass())) {
        try {
            return method.invoke(this, args);
        } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
        }
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
}
// MapperProxy#cachedMapperMethod
private MapperMethod cachedMapperMethod(Method method) {
    // 每一个MapperProxy实例都会对MapperMethod对象进行缓存操作
    MapperMethod mapperMethod = methodCache.get(method);
    if (mapperMethod == null) {
        mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
        methodCache.put(method, mapperMethod);
    }
    return mapperMethod;
}
```

跟踪mapperMethod.execute(sqlSession, args)：

```java
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
        case INSERT: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.insert(command.getName(), param));
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(command.getName(), param);
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("...");
    }
    if (result == null && method.getReturnType().isPrimitive() 
        && !method.returnsVoid()) {
        throw new BindingException("...");
    }
    return result;
}
```

UPDATE、DELETE、INSERT最后都会转化为Executor#update执行：

```java
// SqlSession#insert(java.lang.String, java.lang.Object)
public int update(String statement, Object parameter) {
    try {
        dirty = true;
        MappedStatement ms = configuration.getMappedStatement(statement);
        return executor.update(ms, wrapCollection(parameter));
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
// Executor#update
public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource())
        .activity("executing an update").object(ms.getId());
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    // 清除一级缓存
    clearLocalCache();
    return doUpdate(ms, parameter);
}
```













































































































