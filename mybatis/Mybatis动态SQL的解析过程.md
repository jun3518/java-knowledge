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
    select 
    	* 
    from 
    	<include refid="tableName"/> 
    where
    	id = #{id}
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
    	// 解析<include>标签
    if (source.getNodeName().equals("include")) {
      	// 查找refid属性指向的<sql>节点，返回的是其深克隆的Node对象
      Node toInclude = findSqlFragment(getStringAttribute(source, "refid"), 
                                       variablesContext);
        // 解析<include>节点下的<property>节点，将得到的键值对添加到variablesContext中，并形成
        // 新的Properties对象返回，用于替换占位符
      Properties toIncludeContext = getVariablesContext(source, variablesContext);
      	// 递归处理<include>节点，在<sql>节点中可能会使用<include>引用了其他SQL片段
      applyIncludes(toInclude, toIncludeContext, true);
      if (toInclude.getOwnerDocument() != source.getOwnerDocument()) {
        toInclude = source.getOwnerDocument().importNode(toInclude, true);
      }
        // 将<include>节点替换成<sql>节点
      source.getParentNode().replaceChild(toInclude, source);
      while (toInclude.hasChildNodes()) {
        toInclude.getParentNode().insertBefore(toInclude.getFirstChild(), toInclude);
      }
        // 删除<sql>节点
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
       // 使用之前解析得到的Properties对象替换对应的占位符
      source.setNodeValue(PropertyParser.parse(source.getNodeValue(), variablesContext));
    }
  }
```

对于id为selectList的SQL语句，Mybatis会将其解析为三个部分，即：

```java
else if (source.getNodeType() == Node.ELEMENT_NODE) {
    NodeList children = source.getChildNodes();
    for (int i=0; i<children.getLength(); i++) {
        applyIncludes(children.item(i), variablesContext);
    }
}
```

（1）select  *  from 

（2）\<include refid="tableName"/> 

（3）where id = #{id}

对于（2）类型的节点，Mybatis会解析\<include>节点下的\<property>节点，将得到的键值对添加到variablesContext中，并形成新的Properties对象返回，用于替换占位符。接着递归处理\<include>节点，在\<sql>节点中可能会使用\<include>引用了其他SQL片段。最后删除\<sql>节点。

### 解析SQL

在上述代码中的：

```java
LanguageDriver langDriver = getLanguageDriver(lang);
SqlSource sqlSource = langDriver.createSqlSource(configuration, context, 
                                                 parameterTypeClass);
```

Mybatis默认的LanguageDriver实现类是XMLLanguageDriver。

跟踪代码langDriver.createSqlSource(...)：

```java
public SqlSource parseScriptNode() {
    // 解析SQL内容，如果存在动态SQL节点的元素，递归解析，直到获取到TextSqlNode的List集合
    List<SqlNode> contents = parseDynamicTags(context);
    MixedSqlNode rootSqlNode = new MixedSqlNode(contents);
    SqlSource sqlSource = null;
    if (isDynamic) {
        sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
    } else {
        sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);
    }
    return sqlSource;
}
```

```java
List<SqlNode> parseDynamicTags(XNode node) {
    List<SqlNode> contents = new ArrayList<SqlNode>();
    NodeList children = node.getNode().getChildNodes();
    for (int i = 0; i < children.getLength(); i++) {
        XNode child = node.newXNode(children.item(i));
        // 如果文本Node节点，则将文本内容封装到TextSqlNode中
        if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE || 
            child.getNode().getNodeType() == Node.TEXT_NODE) {
            String data = child.getStringBody("");
            TextSqlNode textSqlNode = new TextSqlNode(data);
            if (textSqlNode.isDynamic()) {
                contents.add(textSqlNode);
                isDynamic = true;
            } else {
                contents.add(new StaticTextSqlNode(data));
            }
        } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) { 
            NodeHandler handler = nodeHandlers(nodeName);
            if (handler == null) {
                throw new BuilderException("...");
            }
            handler.handleNode(child, contents);
            isDynamic = true;
        }
    }
    return contents;
}
```

