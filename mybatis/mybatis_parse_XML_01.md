从下面的代码开始Mybatis源码分析：

```java
public static void main(String[] args) throws IOException {
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
}
```

## SqlSessionFactoryBuilder

SqlSessionFactoryBuilder是利用XML或者Java编码获得资源来构建SqlSessionFactory。通过它可以构建过个SqlSessionFactory。它的作用就是一个构建器，一旦构建了SqlSessionFactory，它的作用也就消失了。

跟踪：SqlSessionFactoryBuilder#build(is)

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    // ...
    //将配置文件的inputStream验证解析Configuration
		XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
    //parser.parse()返回Configuration,并由最后一个方法实例DefaultSqlSessionFactory(config)返回
		return build(parser.parse());
    // ...
```

## XMLConfigBuilder

XMLConfigBuilder类的作用是将mybatis-config.xml的信息转换成Configuration类。

在XMLConfigBuilder#parse()中。

XMLConfigBuilder继承了BaseBuilder，它的成员变量中较为重要的有两个。一个是Configuration类,这个是用来存储mybatis的配置信息的;另一个是XPathParser类，是XPath解析器，用的都是JDK的类包,封装了一下，用来解析XML文件。

```java
private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
}
```

## 解析mybatis-config.xml的元素

org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration(XNode root)：

```java
Properties settings = settingsAsPropertiess(root.evalNode("settings"));
//issue #117 read properties first
propertiesElement(root.evalNode("properties"));
loadCustomVfs(settings);
typeAliasesElement(root.evalNode("typeAliases"));
pluginElement(root.evalNode("plugins"));
objectFactoryElement(root.evalNode("objectFactory"));
objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
reflectionFactoryElement(root.evalNode("reflectionFactory"));
settingsElement(settings);
// read it after objectFactory and objectWrapperFactory issue #631
environmentsElement(root.evalNode("environments"));
databaseIdProviderElement(root.evalNode("databaseIdProvider"));
typeHandlerElement(root.evalNode("typeHandlers"));
mapperElement(root.evalNode("mappers"));
```

### \<settings>

对\<settings>元素进行解析，将其子元素\<setting>封装成Properties对象。

在org.apache.ibatis.builder.xml.XMLConfigBuilder#settingsElement(Properties props)设置解析好的\<setting>，如果没有相关的\<setting>，则，则取系统默认值。

```java
configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
    configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
    configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));
    configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));
    configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
    configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), true));
    configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
    configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));
    configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));
    configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));
    configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));
    configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));
    configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));
    configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));
    configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));
    configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));
    configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
    configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));
    configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));
    configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
    configuration.setLogPrefix(props.getProperty("logPrefix"));
    configuration.setLogImpl(resolveClass(props.getProperty("logImpl")));
    configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));
```



### \<properties>

如果指定了属性resource或url（两者只能选择其一），Mybatis会将其封装到Properties对象中，resource或url中定义的内容会覆盖\<property>元素的内容（即resource或url定义的内容优先级高于\<property>属性）。

然后将其存入到Configuration对象的variables属性中。

### \<typeAliases>

1、如果指定了\<package>子元素，那么会将获取包下的类，然后获取类相应的Class对象和Class对象的simpleName的全小写字符串。

2、如果指定了\<typeAlias>子元素，那么获取该子元素定义的type的Class对象和alias属性值。

然后将其注册到org.apache.ibatis.type.TypeAliasRegistry#TYPE_ALIASESMap中。

注：在org.apache.ibatis.type.TypeAliasRegistry类的构造方法中，注册了基本数据类型及包装类和常用的JDK类的别名。

### \<plugins>

Mybatis会解析\<plugins>的子元素\<plugin>，并将其实例化后存入到org.apache.ibatis.session.Configuration#interceptorChain集合中。

注：插件可以令Mybatis具有扩展性，后面将会重点讲解。

### \<environments>

