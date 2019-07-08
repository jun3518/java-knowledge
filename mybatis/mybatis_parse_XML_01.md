从下面的代码开始Mybatis源码分析：

```java
public static void main(String[] args) throws IOException {
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
}
```

## SqlSessionFactoryBuilder

SqlSessionFactoryBuilder是利用XML或者Java编码获得资源来构建SqlSessionFactory。通过它可以构建过个SqlSessionFactory。它的作用就是一个构建器，一旦构建了SqlSessionFactory，它的作用也就消失了.

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

