## mybatis-config.xml配置文件解析

Mybatis读取全局配置文件mybatis-config.xml的入口是SqlSessionFactoryBuilder#build()方法。

```java
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
    return build(parser.parse());
}
```

在build()方法中，通过传入的全局配置文件mybatis-config.xml的IO流构建一个XMLConfigBuilder对象，然后通过XMLConfigBuilder对象的parse()方法完成解析。

XMLConfigBuilder是BaseBuilder的子类之一，作用是具体的建造者的角色，负责解析mybatis-config.xml配置文件，其核心字段如下：

```java
public class XMLConfigBuilder extends BaseBuilder {
	// 标识是否已经解析过mybatis-config.xml配置文件
    private boolean parsed;
    // 用于解析mybatis-config.xml的XPathParser对象
    private final XPathParser parser;
    // 标识<environment>配置的名称，默认读取<environment>标签的default属性
    private String environment;
    // localReflectorFactory负责创建和缓存Reflector对象
    private final ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();
}
```

XMLConfigBuilder的基类是BaseBuilder，BaseBuilder是相关配置文件如Mapper配置文件的解析类的基类。其中BaseBuilder类中有三个属性：

```java
public abstract class BaseBuilder {
    // Configuration是Mybatis的核心对象,Mybatis运行时的配置信息都会存在此对象中
    protected final Configuration configuration;
    // 类别名存储对象，存储Mybatis默认的别名和用户自定义的别名
    protected final TypeAliasRegistry typeAliasRegistry;
    // TypeHandler存储对象，存储了Mybatis定义的TypeHandler和用户自定义的TypeHandler
    protected final TypeHandlerRegistry typeHandlerRegistry;
}
```

注：在XMLConfigBuilder对象的构造方法中，会实例化一个Configuration对象，并通过基类的构造函数BaseBuilder()将其传到BaseBuilder对象中。

继续跟踪XMLConfigBuilder#parse()内代码：

```java
// XMLConfigBuilder#parseConfiguration()方法：
private void parseConfiguration(XNode root) {
    // 解析<properties>节点
    propertiesElement(root.evalNode("properties"));
    // 解析<settings>节点
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    // 设置vfsImpl字段
    loadCustomVfs(settings);
    // 解析<typeAliases>节点
    typeAliasesElement(root.evalNode("typeAliases"));
    // 解析<plugins>节点
    pluginElement(root.evalNode("plugins"));
    // 解析<objectFactory>节点
    objectFactoryElement(root.evalNode("objectFactory"));
    // 解析<objectWrapperFactory>节点
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    // 解析<reflectorFactory>节点
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    // 将settings值设置到Configuration中
    settingsElement(settings);
    // 解析<environments>节点
    environmentsElement(root.evalNode("environments"));
    // 解析<databaseIdProvider>节点
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    // 解析<typeHandlers>节点
    typeHandlerElement(root.evalNode("typeHandlers"));
    // 解析<mappers>节点
    mapperElement(root.evalNode("mappers"));
}
```



### 解析\<properties>节点

XMLConfigBuilder.propertiesElement()方法会解析mybatis-config.xml配置文件中的\<properties>节点并形成Properties对象，之后将该Properties对象设置到XPathParser和Configuration的variables字段中。在后面的解析过程中，会使用该Properties对象中的信息替换占位符。

```java
private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
        // 解析<properties>的子节点（标签<property>标签）的name和value属性，并记录到Properties中
        Properties defaults = context.getChildrenAsProperties();
        // 解析<properties>的resource和url属性，这两个属性用于确定properties配置文件的位置
        String resource = context.getStringAttribute("resource");
        String url = context.getStringAttribute("url");
        // resource和url属性不能同时存在，否则会抛出异常
        if (resource != null && url != null) {
            throw new BuilderException("...");
        }
        // 加载resource属性或url指定的properties文件
        if (resource != null) {
            defaults.putAll(Resources.getResourceAsProperties(resource));
        } else if (url != null) {
            defaults.putAll(Resources.getUrlAsProperties(url));
        }
        // 与Configuration对象中的variables集合合并
        Properties vars = configuration.getVariables();
        if (vars != null) {
            defaults.putAll(vars);
        }
        // 更新XPathParser和Configuration的variables字段
        parser.setVariables(defaults);
        configuration.setVariables(defaults);
    }
}
```

注：当同时存在\<property>标签元素和\<properties>属性resource或url属性，如果有相同的属性时，resource或url中定义的属性会覆盖\<property>定义的属性，因为\<property>先解析，先解析的属性会被后解析的属性覆盖。

#### \<properties>示例演示

```xml
<properties resource="db.properties">
    <property name="jdbc.username" value="zhangsan"/>
    <property name="jdbc.password" value="123456"/>
</properties>
```



### 解析\<settings>节点

XMLConfigBuilder.settingsAsProperties()方法负责解析\<settings>节点，在\<settings>节点下的配置是Mybatis全局性的配置，它们会改变Mybatis的运行时行为。

在Mybatis初始化时，这些全局配置信息都会被记录到Configuration对象的对应属性中。

代码逻辑简单，代码略。

#### \<settings>示例演示

```xml
<settings>
    <!-- 忽略下划线“_”，并将属性根据下划线转化为驼峰规则-->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```



### 解析\<typeAliases>

XMLConfigBuilder#typeAliasesElement()方法负责解析\<typeAliases>节点及其子节点，并通过TypeAliasRegistry完成别名的注册：

```java
private void typeAliasesElement(XNode parent) {
    if (parent != null) {
        // 处理全部子节点
        for (XNode child : parent.getChildren()) {
            // 处理<package>节点
            if ("package".equals(child.getName())) {
                // 获取指定的包名
                String typeAliasPackage = child.getStringAttribute("name");
                // 通过TypeAliasRegistry扫描指定包中所有的类，并即系@Alias注解，完成别名的注册
                configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
            } else { // 处理<typeAliases>节点
                // 获取指定的别名
                String alias = child.getStringAttribute("alias");
                // 获取别名对应的类型
                String type = child.getStringAttribute("type");
                try {
                    Class<?> clazz = Resources.classForName(type);
                    if (alias == null) {
                        // 扫描@Alias注解，完成注册
                        typeAliasRegistry.registerAlias(clazz);
                    } else {
                        // 注册别名
                        typeAliasRegistry.registerAlias(alias, clazz);
                    }
                } catch (ClassNotFoundException e) {
                    throw new BuilderException("...");
                }
            }
        }
    }
}
//在没有指定别名的情况下执行此段代码
public void registerAlias(Class<?> type) {
    // 获取Class的简单类名
    String alias = type.getSimpleName();
    // 获取Class的Alias别名注解
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    // 别名优先级高于Class的简单类名
    if (aliasAnnotation != null) {
        alias = aliasAnnotation.value();
    } 
    // 注册别名
    registerAlias(alias, type);
}
// 注册别名
public void registerAlias(String alias, Class<?> value) {
    // 将别名转化为小写
    String key = alias.toLowerCase(Locale.ENGLISH);
    // 如果该别名已存在、别名不为空，且和当前解析的别名相等，则抛异常
    // 如果该别名已存在、别名不为空，且和当前别名不想等，则将原来的别名覆盖
    // 如果该别名已存在、别名为空，则将为空的别名覆盖
    if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null 
        && !TYPE_ALIASES.get(key).equals(value)) {
      throw new TypeException("...");
    }
    TYPE_ALIASES.put(key, value);
  }
```

（1）类的别名优先级：配置文件制定的别名 > 别名注解制定的优先级 > 类的简单名称。

（2）别名都会转化为小写。

（3）如果该别名已存在、别名不为空，且和当前解析的别名相等，则抛异常；如果该别名已存在、别名不为空，且和当前别名不想等，则将原来的别名覆盖；如果该别名已存在、别名为空，则将为空的别名覆盖。

#### \<typeAliases>示例演示

```xml
<typeAliases>
    <!-- 
    package#name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写））
	如果想为某个类定义别名，则可以使用org.apache.ibatis.type.Alias注解定义
  	-->
    <package name="com.mybatis.bean"/>
    <typeAlias type="com.mybatis.bo.UserBean" alias="userBean"/>
</typeAliases>
```



### 解析\<plugins>节点

插件是Mybatis提供的扩展机制之一，用户可以通过添加自定义插件在SQL语句执行过程中的某一点进行拦截。Mybatis中的自定义插件只需要实现Interceptor接口，并通过注解指定想要拦截的方法签名即可。

```java
private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            String interceptor = child.getStringAttribute("interceptor");
            // 获取<plugin>节点下<properties>配置的信息，并形成Properties对象
            Properties properties = child.getChildrenAsProperties();
            // 经过TypeAliasRegistry解析别名之后。实例化Interceptor对象
            Interceptor interceptorInstance = 
                (Interceptor) resolveClass(interceptor).newInstance();
            interceptorInstance.setProperties(properties);
            configuration.addInterceptor(interceptorInstance);
        }
    }
}
```

Mybatis会将解析的Interceptor实现类添加到Configuration#interceptorChain成员属性中。interceptorChain底层是一个ArrayList。

注：\<plugins>属于Mybatis扩展功能，会单独写新的文章分析说明。



### 解析\<objectFactory>、\<objectWrapperFactory>节点

\<objectFactory>、\<objectWrapperFactory>节点是Mybatis提供的扩展功能的节点，这篇文章不会讲解。



### 解析\<environments>节点

在实际生产中，同一项目可能分为开发、测试和生产多个不同的环境，每个环境的配置可能也不尽相同。Mybatis可以配置多个\<environment>节点，每个\<environment>节点对应一种环境的配置。

尽管可以配置多个环境，但是每个SqlSessionFactory实例只能选择其一。

XMLConfigBuilder#environmentsElement()方法负责解析\<environments>的相关配置，它会根据XMLConfigBuilder.environment字段确定要使用的\<environment>配置，之后创建对应的TransactionFactory和DataSource对象，并封装进Environment对象中。

```java
private void environmentsElement(XNode context) throws Exception {
    if (context != null) {
        // 未指定XMLConfigBuilder.environment字段，则使用default属性指定<environment>
        if (environment == null) {
            environment = context.getStringAttribute("default");
        }
        // 遍历子节点，即<environment>节点
        for (XNode child : context.getChildren()) {
            // 与XMLConfigBuilder.environment字段匹配
            String id = child.getStringAttribute("id");
            if (isSpecifiedEnvironment(id)) {
            // 创建TransactionFactory，具体实现是先通过TypeAliasRegistry解析别名后，实例化
            // TransactionFactory
                TransactionFactory txFactory = 
                    transactionManagerElement(child.evalNode("transactionManager"));
                // 创建DataSourceFactory和DataSource
                DataSourceFactory dsFactory = 
                    dataSourceElement(child.evalNode("dataSource"));
                DataSource dataSource = dsFactory.getDataSource();
               // 将Environment对象记录到Configuration.environment字段中
                Environment.Builder environmentBuilder = new Environment.Builder(id)
                    .transactionFactory(txFactory)
                    .dataSource(dataSource);
                configuration.setEnvironment(environmentBuilder.build());
            }
        }
    }
}
```

在Configuration对象初始化时，会在构造函数中注册一些系统默认的别名：

```java
public Configuration() {
    // TransactionFactory类型的类的别名注册
    typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
    typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
    // DataSourceFactory类型的类的别名注册
    typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
    typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
    typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
    // ...
}
```

#### TransactionFactory：

在上述代码中的environment变量，如果在XMLConfigBuilder对象初始化时（即SqlSessionFactoryBuilder初始化时传入）传入，则优先级最高。如果environment为null（没有传入），则解析获取mybatis-config.xml文件中配置的environment。

（1）JdbcTransactionFactory：使用JDBC的事务管理机制：即利用java.sql.Connection对象完成对事务的提交（commit()）、回滚（rollback()）、关闭（close()）等。

（2）ManagedTransactionFactory：使用MANAGED的事务管理机制：这种机制MyBatis自身不会去实现事务管理，而是让程序的容器如（JBOSS，Weblogic）来实现对事务的管理。

注：可以通过实现TransactionFactory接口，并在Mybatis注册该接口实现类的别名，而实现自定义事务的扩展。

#### DataSourceFactory和DataSource

（1）JndiDataSourceFactory：JndiDataSourceFactory 依赖 JNDI 服务器中获取用户配置的 DataSource。

（2）PooledDataSourceFactory ：PooledDataSourceFactory 主要用来创建 PooledDataSource 对象，它继承了 UnpooledDataSource 类，设置 DataSource 参数的方法复用UnpooledDataSource 中的 setProperties 方法，只是数据源返回的是  PooledDataSource 对象而已。

（3）UnpooledDataSourceFactory ：UnpooledDataSourceFactory 主要用来创建 UnpooledDataSource 对象，它会在构造方法中初始化 UnpooledDataSource 对象，并在 setProperties 方法中完成对 UnpooledDataSource 对象的配置。

（4）UnpooledDataSource：UnpooledDataSource 不使用连接池来创建数据库连接，每次获取数据库连接时都会创建一个新的连接进行返回。

（5）PooledDataSource：PooledDataSource是Mybatis实现的一个简单连接池，是UnpooledDataSource 来实现的。

#### \<environments>示例演示

```xml
<environments default="mysql">
    <environment id="mysql">
        <transactionManager type="JDBC">
        </transactionManager>
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}" />
            <property name="url" value="${jdbc.url}" />
            <property name="username" value="${jdbc.username}" />
            <property name="password" value="${jdbc.password}" />
        </dataSource>
    </environment>

    <environment id="oracle">
        <transactionManager type="JDBC" />
        <dataSource type="POOLED">
            <property name="driver" value="${orcl.driver}" />
            <property name="url" value="${orcl.url}" />
            <property name="username" value="${orcl.username}" />
            <property name="password" value="${orcl.password}" />
        </dataSource>
    </environment>
</environments>
```

在上述的配置代码中，\<environments default="mysql">指定了Mybatis在运行时使用的是\<environment id="mysql">配置。其中\<transactionManager type="JDBC">表示使用JdbcTransactionFactory事务工厂；\<dataSource type="POOLED">表示使用PooledDataSourceFactory连接池工厂。



### 解析\<databaseIdProvider>节点

Mybatis不能像Hibernate一样直接屏蔽多种数据库产品在SQL语言支持方面的差异。但是在mybatis-config.xml配置文件中，通过\<databaseIdProvider>节点定义所有支持的数据库产品的databaseId，然后在映射配置文件中定义SQL语句节点时，通过databaseId指定该SQL语句应用的数据库产品，这样也可以实现类似的功能。

XMLConfigBuilder#databaseIdProviderElement()方法负责解析\<databaseIdProvider>节点，并创建指定的DatabaseIdProvider对象。DatabaseIdProvider会返回databaseId值，并将其设置到Configuration对象的databaseId属性中，Mybatis会根据databaseId选择合适的SQL进行执行。

```java
private void databaseIdProviderElement(XNode context) throws Exception {
    DatabaseIdProvider databaseIdProvider = null;
    if (context != null) {
        String type = context.getStringAttribute("type");
        // 为了保证兼容性，修改type取值
        if ("VENDOR".equals(type)) {
            type = "DB_VENDOR";
        }
        // 解析相关配置信息
        Properties properties = context.getChildrenAsProperties();
        // 创建DatabaseIdProvider对象
        databaseIdProvider = (DatabaseIdProvider) resolveClass(type).newInstance();
        // 配置DatabaseIdProvider，完成初始化
        databaseIdProvider.setProperties(properties);
    }
    Environment environment = configuration.getEnvironment();
    if (environment != null && databaseIdProvider != null) {
        // 通过前面确定的DataSource获取databaseId，并记录到Configuration.databaseId字段中
        String databaseId =  
            databaseIdProvider.getDatabaseId(environment.getDataSource());
        configuration.setDatabaseId(databaseId);
    }
}
```

在Configuration对象初始化时，会在构造函数中注册一些系统默认的别名：

```java
public Configuration() {
	typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);
    // ...
}
```

Mybatis默认的DatabaseIdProvider接口实现有VendorDatabaseIdProvider。可以通过实现DatabaseIdProvider接口并将该实现配置到Mybatis中实现扩展。

#### \<databaseIdProvider>示例演示

```xml
<databaseIdProvider type="DB_VENDOR">
    <!-- 为不同的数据库产品起别名 -->
    <property name="MySQL" value="mysql"/>
    <property name="Oracle" value="oracle"/>
    <property name="SQL Server" value="sqlserver"/>
</databaseIdProvider>
```

以DB_VENDOR（VendorDatabaseIdProvider）为例，在VendorDatabaseIdProvider#getDatabaseId()方法中，Mybatis会获取当前环境中运行的数据库产品是哪一个，如果为null，则说明没有匹配的数据库产品。

```java
public String getDatabaseId(DataSource dataSource) {
    if (dataSource == null) {
        throw new NullPointerException("dataSource cannot be null");
    }
    return getDatabaseName(dataSource);
}

private String getDatabaseName(DataSource dataSource) throws SQLException {
    String productName = getDatabaseProductName(dataSource);
    // this.properties是配置文件中配置的参数
    if (this.properties != null) {
        for (Map.Entry<Object, Object> property : properties.entrySet()) {
            if (productName.contains((String) property.getKey())) {
                return (String) property.getValue();
            }
        }
        // 没有匹配，返回null
        return null;
    }
    // 如果没有配置数据库产品参数，则直接返回当前环境的数据库产品名称
    return productName;
}
```



### 解析\<mappers>节点

\<mappers>节点的解析会在新的文章中会继续分析。暂略。



