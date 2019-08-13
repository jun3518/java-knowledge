## SqlSessionFactoryBuilder

Mybatis通过读取mybatis-config.xml文件，将解析后的信息加载到Configuration对象中，Mybatis在运行时根据Configuration对象的信息进行相应的操作。

SqlSessionFactoryBuilder是Mybatis读取配置mybatis-config.xml文件的入口：

```java
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
    return build(parser.parse());
  }
```

XMLConfigBuilder是负责解析mybatis-config.xml文件的对象，它继承自BaseBuilder类。BaseBuilder类说明：

```java
public abstract class BaseBuilder {
    // Configuration是Mybatis的核心对象,Mybatis运行时的配置信息都会存在此对象中
    protected final Configuration configuration;
    // 类别名存储对象，存储Mybatis默认的别名和用户自定义的别名
    protected final TypeAliasRegistry typeAliasRegistry;
    // TypeHandler存储对象，存储了Mybatis定义的TypeHandler和用户自定义的TypeHandler
    protected final TypeHandlerRegistry typeHandlerRegistry;
    
    // BaseBuilder的构造函数
    public BaseBuilder(Configuration configuration) {
    this.configuration = configuration;
    this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
    this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
  }
}
```

其中TypeHandlerRegistry和TypeAliasRegistry对象都是Configuration对象初始化时创建的。

BaseBuilder的继承关系图如下：

![](./images/BaseBuilder继承关系图.jpg)

BaseBuilder的这些子类后面会进行说明。

## XMLConfigBuilder

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

### 解析\<settings>节点

XMLConfigBuilder.settingsAsProperties()方法负责解析\<settings>节点，在\<settings>节点下的配置是Mybatis全局性的配置，它们会改变Mybatis的运行时行为。

在Mybatis初始化时，这些全局配置信息都会被记录到Configuration对象的对应属性中。

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
```

### 解析\<typeHandlers>节点

```java
private void typeHandlerElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            if ("package".equals(child.getName())) {
                String typeHandlerPackage = child.getStringAttribute("name");
                typeHandlerRegistry.register(typeHandlerPackage);
            } else {
                String javaTypeName = child.getStringAttribute("javaType");
                String jdbcTypeName = child.getStringAttribute("jdbcType");
                String handlerTypeName = child.getStringAttribute("handler");
                Class<?> javaTypeClass = resolveClass(javaTypeName);
                JdbcType jdbcType = resolveJdbcType(jdbcTypeName);
                Class<?> typeHandlerClass = resolveClass(handlerTypeName);
                if (javaTypeClass != null) {
                    if (jdbcType == null) {
                        typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);
                    } else {
                        typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);
                    }
                } else {
                    typeHandlerRegistry.register(typeHandlerClass);
                }
            }
        }
    }
}
```

XMLConfigBuilder.typeHandlerElement()方法负责解析\<typeHandlers>节点，并通过TypeHandlerRegistry对象完成TypeHandler的注册。

