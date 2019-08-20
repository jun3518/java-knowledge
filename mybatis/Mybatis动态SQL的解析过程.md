## 概述

MyBatis 的强大特性之一便是它的动态 SQL。如果你有使用 JDBC 或其它类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句的痛苦。例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

虽然在以前使用动态 SQL 并非一件易事，但正是 MyBatis 提供了可以被用在任意 SQL 映射语句中的强大的动态 SQL 语言得以改进这种情形。

MyBatis 采用功能强大的基于 OGNL 的表达式来实现Mybatis的动态SQL功能。

Mybatis提供了以下的动态SQL标签元素：

trim、where、set、foreach、if、choose、when、otherwise、bind。

## Mybatis对SQL节点\<SELECT>、\<UPDATE>、\<INSERT>、\<DELETE>的处理流程

```java
buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
```

在XMLMapperBuilder#buildStatementFromContext(List\<XNode>)方法中，MyBatis对SQL节点\<INSERT>、\<DELETE>、\<UPDATE>、\<SELECT>四个SQL节点进行解析。

```java
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
        final XMLStatementBuilder statementParser = new 
            XMLStatementBuilder(configuration, builderAssistant, context, 
                                requiredDatabaseId);
        try {
            statementParser.parseStatementNode();
        } catch (IncompleteElementException e) {
            configuration.addIncompleteStatement(statementParser);
        }
    }
}
```

在上述代码中的statementParser.parseStatementNode();中， 封装了对单个SQL节点的处理逻辑，一下代码是只保留和SQL节点解析相关的代码，其他的代码不在讨论的范围内：

```java
public void parseStatementNode() {
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    // 在解析SQL语句之前，先处理其中的<include>节点
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    // 处理<selectKey>节点
    processSelectKeyNodes(id, parameterTypeClass, langDriver);
    
    // 完成<selectKey>和<include>节点的解析和移除后，进行动态SQL节点的解析
    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
}
```

### 在解析SQL语句之前，先处理其中的\<include>节点

```xml
<!--sql片段-->
<sql id="tableName">
    user
</sql>
<!--sql语句-->
<select id="selectList" resultMap="com.mybatis.bo.User">
    select * from <include refid="tableName"/>
</select>
```

在XMLIncludeTransformer#applyIncludes(Node)方法中，先从Configuration中获取全局Properties属性集，然后调用XMLIncludeTransformer#applyIncludes(Node, Properties, boolean)方法进行解析操作：

```java
public void applyIncludes(Node source) {
    Properties variablesContext = new Properties();
    Properties configurationVariables = configuration.getVariables();
    if (configurationVariables != null) {
        variablesContext.putAll(configurationVariables);
    }
    applyIncludes(source, variablesContext, false);
}
```

```java
private void applyIncludes(Node source, final Properties variablesContext, boolean included) {
    if (source.getNodeName().equals("include")) {
      Node toInclude = findSqlFragment(getStringAttribute(source, "refid"), 
                                       variablesContext);
      Properties toIncludeContext = getVariablesContext(source, variablesContext);
      applyIncludes(toInclude, toIncludeContext, true);
      if (toInclude.getOwnerDocument() != source.getOwnerDocument()) {
        toInclude = source.getOwnerDocument().importNode(toInclude, true);
      }
      source.getParentNode().replaceChild(toInclude, source);
      while (toInclude.hasChildNodes()) {
        toInclude.getParentNode().insertBefore(toInclude.getFirstChild(), toInclude);
      }
      toInclude.getParentNode().removeChild(toInclude);
    } else if (source.getNodeType() == Node.ELEMENT_NODE) {
      if (included && !variablesContext.isEmpty()) {
        // replace variables in attribute values
        NamedNodeMap attributes = source.getAttributes();
        for (int i = 0; i < attributes.getLength(); i++) {
          Node attr = attributes.item(i);
          attr.setNodeValue(PropertyParser.parse(attr.getNodeValue(), variablesContext));
        }
      }
      NodeList children = source.getChildNodes();
      for (int i = 0; i < children.getLength(); i++) {
        applyIncludes(children.item(i), variablesContext, included);
      }
    } else if (included && source.getNodeType() == Node.TEXT_NODE
        && !variablesContext.isEmpty()) {
      // replace variables in text node
      source.setNodeValue(PropertyParser.parse(source.getNodeValue(), variablesContext));
    }
  }
```



