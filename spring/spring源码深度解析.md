# 第2章 容器的基本实现

## 2.4 Spring的结构组成

### 2.4.2 核心类介绍

#### 1. DefaultListableBeanFactory

XmlBeanFactory继承自DefaultListableBeanFactory，而DefaultListableBeanFactory是整个bean加载的核心部分，是Spring注册及加载bean的默认实现。而对于XmlBeanFactory与DefaultListableBeanFactory不同的地方是在XMLBeanFactory中使用了自定义的XML读取器XmlBeanDefinitionReader，实现了个性化的BeanDefinitionReader读取。DefaultListableBeanFactory继承了AbstractAutowireCapableBeanFactory并实现了ConfigurableListableBeanFactory以及BeanDefinitionRegistry接口，以下是ConfigurableListableBeanFactory的层次结构图：

![](./images/ConfigurableListableBeanFactory的层次结构图.png)

（1）AliasRegistry：定义对alias的简单增删改等操作。

（2）SimpleAliasRegistry：主要使用map作为alias的缓存，并对接口AliasRegistry进行实现。

（3）SingletonBeanRegistry：定义对单例的注册及获取。

（4）BeanFactory：定义获取bean及bean的各种属性。

（5）HierarchicalBeanFactory：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持。

（6）BeanDefinitionRegistry：定义对BeanDefinition的各种增删改操作。

（7）FactoryBeanRegistrySupport：在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。

（8）ConfigurableBeanFactory：提供配置Factory的各种方法。

（9）ListableBeanFactory：根据各种条件获取bean的配置清单。

（10）AbstractBeanFactory：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。

（11）AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的后处理器。

（12）AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并对接口AutowireCapableBeanFactory进行实现。

（13）ConfigurableListableBeanFactory：BeanFactory配置清单，指定忽略类型及接口等。

（14）DefaultListableBeanFactory综合上面所有功能，主要是对Bean注册后的处理。

XmlBeanFactory对DefaultListableBeanFactory类进行了扩展，主要用于从XML文档中读取BeanDefinition。对于注册及获取Bean都是使用父类DefaultListableBeanFactory继承的方法去实现，而唯独与父类不同的个性化实现就是增加了XmlBeanDefinitionReader类型的reader属性。在XmlBeanFactory中主要使用reader属性对资源文件进行读取和注册。

#### 2. XmlBeanDefinitionReader

XML配置文件的读取是Spring中重要的功能，因为Spring的大部分功能都是以配置作为切入点的。XmlBeanDefinitionReader和父类及属性的功能：

（1）ResourceLoader：定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource。

（2）BeanDefinitionReader：主要定义资源文件读取并转换为BeanDefinition的各个功能。

（3）EnvironmentCapable：定义获取Environment方法。

（4）DocumentLoader：定义从资源文件加载到转换为Document的功能。

（5）AbstractBeanDefinitionReader：对EnvironmentCapable、BeanDefinitionReader类定义的功能进行实现。

（6）BeanDefinitionDocumentReader：定义读取Document并注册BeanDefinition功能。

（7）BeanDefinitionParserDelegate：定义解析Element的功能方法。

![](./images/配置文件读取的相关类图.png)

XML配置文件读取的大致流程：

（1）通过继承AbstractBeanDefinitionReader中的方法，来使用ResourceLoader将资源文件路径转化为对应的Resource文件。

（2）通过DocumentLoader对Resource文件进行转化，将Resource文件转化为Document文件。

（3）通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析。

## 2.5 容器的基础XmlBeanFactory

![](./images/XmlBeanFactory初始化时序图.png)

## 2.8 解析及注册BeanDefinitions

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    // 使用DefaultBeanDefinitionDocumentReader实例化BeanDefinitionDocumentReader
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
// ...
protected void doRegisterBeanDefinitions(Element root) {
    // 专门处理解析
    BeanDefinitionParserDelegate parent = this.delegate;
    this.delegate = createDelegate(getReaderContext(), root, parent);

    if (this.delegate.isDefaultNamespace(root)) {
        // 处理profile属性
        String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
        if (StringUtils.hasText(profileSpec)) {
            String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
                profileSpec, 
                BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
            if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)){
                return;
            }
        }
    }
    // 解析钱处理，留给子类实现
    preProcessXml(root);
    // 解析真正的逻辑
    parseBeanDefinitions(root, this.delegate);
    // 解析后处理，留给子类实现
    postProcessXml(root);

    this.delegate = parent;
}
```

### 2.8.2 解析并注册BeanDefinition

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate){
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;
                if (delegate.isDefaultNamespace(ele)) {
                    // 对bean的处理，Spring默认的配置
                    parseDefaultElement(ele, delegate);
                }
                else {
                    // 对bean的处理，用户自定义的配置
                    delegate.parseCustomElement(ele);
                }
            }
        }
    }
    else {
        delegate.parseCustomElement(root);
    }
}
```

Spring的XML配置里面有两大类Bean声明：

```xml
<!-- 默认的 -->
<bean id="test" class="test.TestBean"/>
<!-- 自定义 -->
<tx:annotation-drivern>
```

默认的配置Spring知道该怎么做，但是如果是自定义的，那就需要用户实现一些接口及配置了。对于根节点或者子节点如果是默认命名空间的话，则采用parseDefaultElement方法进行解析，否则使用delegate.parseCustomElement方法对自定义命名空间进行解析。

判断是否默认命名空间还是自定义命名空间的办法是使用node.getNamespaceURI()获取命名空间，并与Spring中固定的命名空间进行比对，如果一致则认为是默认，否则就认为是自定义。

# 第3章 默认标签的解析

默认标签的解析是在parseDefaultElement函数中进行的，函数中的功能逻辑是分别对4种不同标签（import、alias、bean和beans）做了不同的处理。

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
   	// 对import标签的处理
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    }
    // 对alias标签的处理
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    }
    // 对bean标签的处理
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    // 对beans标签的处理
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        doRegisterBeanDefinitions(ele);
    }
}
```

## 3.1 bean标签的解析及注册

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate){
    // 委托BeanDefinitionParserDelegate类的parseBeanDefinitionElement方法进行元素解析
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        // 若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析。
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        // 解析完成后，需要对解析后的bdHolder进行注册
        BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder,
                                                       getReaderContext().getRegistry());
    }
        // 发出响应时间，通知相关的监听器
        getReaderContext().fireComponentRegistered(
            new BeanComponentDefinition(bdHolder));
    }
}
```



（1）首先委托BeanDefinitionParserDelegate类的parseBeanDefinitionElement方法进行元素解析，返回BeanDefinitionHolder类型的实例bdHolder，经过这个方法后，bdHolder实例已经包含配置文件中配置的各种属性了，如：class、name、id等。

（2）当返回的bdHolder不为空的情况下，若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析。

（3）解析完成后，需要对解析后的bdHolder进行注册，同样注册操作委托给了BeanDefinitionReaderUtils.registerBeanDefinition方法。

（4）发出响应时间，通知相关的监听器，这个bean已经加载完成了。

![](./images/bean标签解析及注册时序图.png)

### 3.1.1 解析BeanDefinition

在BeanDefinitionParserDelegate#parseBeanDefinitionElement(Element, BeanDefinition)中是对默认标签解析的全过程：

（1）提取元素中的id以及name属性。

（2）进一步解析其他所有属性并统一封装至GenericBeanDefinition类型的实例中。

（3）如果检测到bean没有指定beanName，那么使用默认规则为此Bean生成beanName。

（4）将获取到的信息封装到BeanDefinitionHolder的实例中。

#### 1. 创建用于属性承载的BeanDefinition

BeanDefinition是一个借口，在Spring中存在三种实现：RootBeanDefinition、ChildBeanDefinition以及GenericBeanDefinition。三种实现均继承了AbstractBeanDefinition，其中BeanDefinition是配置文件\<bean>元素标签在容器中的内部表示形式。

\<bean>元素标签拥有class、scope、lazy-init等配置属性，BeanDefinition则提供了相应的beanClass、scope、lazyInit属性，BeanDefinition和\<bean>中的属性是一一对应的。其中RootBeanDefinition是最常用的实现类，它对应一般性的\<bean>元素标签。

在配置文件中可以定义父\<bean>和子\<bean>，父\<bean>用RootBeanDefinition表示，而子\<bean>用ChildBeanDefinition表示。

Spring通过BeanDefinition将配置文件中的\<bean>配置信息转换为容器的内部表示，并将这些BeanDefinition注册到BeanDefinitionRegistry中。Spring容器的BeanDefinitionRegistry就像Spring配置信息的内存数据库，主要是以map的形式保存，后续操作直接从BeanDefinitionRegistry中读取配置信息。

![](./images/BeanDefinition及其实现类.jpg)

要解析属性首先要创建用于承载属性的实例，也就是创建GenericBeanDefinition类型的实例。BeanDefinitionParserDelegate#createBeanDefinition的作用就是实现此功能。

#### 2. 解析各种属性

当创建了bean信息的承载实例BeanDefinition后，便可以进行bean信息的各种属性解析了。BeanDefinitionParserDelegate#parseBeanDefinitionAttributes方法是对element所有元素属性进行解析。

#### 3. 解析子元素meta

元数据meta属性的使用：

```xml
<bean id="testBean" class="bean.TestBean">
	<meta key="testStr" value="aaa"/>
</bean>
```

这段代码不会体现在TestBean的属性当中，而是一个额外的声明，当需要使用里面的信息的时候可以通过BeanDefinition的getAttribute(key)方法进行获取。

BeanDefinitionParserDelegate#parseMetaElements方法用于解析子元素meta。

#### 4. 解析子元素lookup-method

#### 5. 解析子元素replaced-method

#### 6. 解析子元素constructor-arg

#### 7. 解析子元素property

#### 8. 解析子元素qualifier

### 3.1.2 AbstractBeanDefinition属性

XML中所有的配置都可以在GenericBeanDefinition的实例类中找到对应的配置。GenericBeanDefinition只是子类实现，而大部分的通用属性都保存在了AbstractBeanDefinition中。

### 3.1.3 解析默认标签中的自定义标签元素

对于BeanDefinitionParserDelegate#decorateBeanDefinitionIfRequired(Element, BeanDefinitionHolder)方法，当Spring中的bean使用的是默认标签配置，但其中的子元素却使用了自定义的配置时，此函数就会起作用。

```xml
<bean id="test" class="test.MyClass">
	<mybean:user username="aaa"/>
</bean>
```

跟踪源码：

```java
public BeanDefinitionHolder decorateBeanDefinitionIfRequired(
			Element ele, BeanDefinitionHolder definitionHolder, 
    		BeanDefinition containingBd) {

    BeanDefinitionHolder finalDefinition = definitionHolder;
    NamedNodeMap attributes = ele.getAttributes();
    // 遍历所有的属性，看看是否有适合用于修饰的属性
    for (int i = 0; i < attributes.getLength(); i++) {
        Node node = attributes.item(i);
        finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
    }
    NodeList children = ele.getChildNodes();
    // 遍历所有的子节点，看看是否有适用于修饰的子元素
    for (int i = 0; i < children.getLength(); i++) {
        Node node = children.item(i);
        if (node.getNodeType() == Node.ELEMENT_NODE) {
            finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
        }
    }
    return finalDefinition;
}
public BeanDefinitionHolder decorateIfRequired(
    Node node, BeanDefinitionHolder originalDef, BeanDefinition containingBd) {
    // 获取自定义标签的命名空间
    String namespaceUri = getNamespaceURI(node);
    // 对于非默认标签进行修饰
    if (!isDefaultNamespace(namespaceUri)) {
        // 根据命名空间找到对应的处理器
        NamespaceHandler handler = 
            this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
        if (handler != null) {
            // 进行修饰
            return handler.decorate(node, originalDef, new ParserContext(
                this.readerContext, this, containingBd));
        }
    }
    return originalDef;
}
```

上述的代码是：首先获取属性或者元素的命名空间，以此来判断该元素或者属性是否适用于自定义标签的解析条件，找出自定义类型所对应的NamespaceHandler并进行进一步解析。

对于decorateBeanDefinitionIfRequired方法，对于程序默认的标签的处理死直接略过的，因为默认的标签到这里就已经处理完了。在方法中实现了寻找自定义标签并根据自定义标签寻找命名空间处理器，并进行进一步的解析。

### 3.1.4 注册解析的BeanDefinition

```java
public static void registerBeanDefinition( BeanDefinitionHolder definitionHolder, 
                   BeanDefinitionRegistry registry) throws BeanDefinitionStoreException {
	// 使用beanName做唯一标识注册
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
    // 注册所有的别名
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}
```

解析的BeanDefinition都会被注册到BeanDefinitionRegistry类型的实例中，而对于BeanDefinition的注册分成了两部分：通过beanName的注册和通过别名的注册。

#### 1. 通过beanname注册BeanDefinition

```java
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {
    // ...
}
```

对于bean的注册处理方式上，主要进行了几个步骤：

（1）对于AbstractBeanDefinition的校验。此处的校验时针对于AbstractBeanDefinition的methodOvrride属性的。

（2）对beanName已经注册的情况的处理。如果设置了不允许bean的覆盖，则需要抛出异常，否则直接覆盖。

（3）加入map缓存。

（4）清除解析之前留下的对应beanName的缓存。

#### 2. 通过别名注册BeanDefinition

```java
public void registerAlias(String name, String alias) {
    synchronized (this.aliasMap) {
        // 如果beanName与alias相同的话，不记录alias，并删除对应的alias
        if (alias.equals(name)) {
            this.aliasMap.remove(alias);
        }
        else {
            // 如果alias不允许被覆盖，则抛出异常
            String registeredName = this.aliasMap.get(alias);
            if (registeredName != null) {
                if (registeredName.equals(name)) {
                    return;
                }
                if (!allowAliasOverriding()) {
                    throw new IllegalStateException("...");
                }
            }
            // 当A->B存在时，若再次出现A->C->B时候，则会抛出异常
            checkForAliasCircle(name, alias);
            this.aliasMap.put(alias, name);
        }
    }
}
```

注册alias的步骤如下：

（1）alias与beanName相同情况处理。若alias与beanName并名称相同，则不需要处理并删除掉原有alias。

（2）alias覆盖处理。若aliasName已经使用并已经指向了另一个beanName，则需要用户的设置进行处理。

（3）alias循环检查。当A->B存在时，若再次出现A->C->B时候，则会抛出异常。

### 3.1.5 通知监听器解析及注册完成

通过代码getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));完成此工作。这里的实现只是为扩展，当程序开发人员需要对注册BeanDefinition事件进行监听时，可以通过注册监听器的方式并将处理逻辑写入监听器中，目前在Spring中并没有对此事件做任何逻辑处理。

## 3.2 alias标签的解析

在对bean进行定义时，除了使用id属性来指定名称之外，为了提供多个名称，可以使用alias标签来指定。而所有的这些名称都指向同一个bean。在某些情况下提供别名非常有用，比如为了让应用的每一个组件能更容易地对公共组件进行引用。

在定义bean时就指定所有的别名并不是总是恰当的。有时候期望能在当前位置为那些在别处定义的bean引入别名。在XML配置文件中，可以单独的\<alias/>元素来完成bean别名的定义。如配置文件中定义了一个JavaBean：

```xml
<bean id="testBean" class="com.TestBean"/>
```

要给这个JavaBean增加别名，以便不同对象来调用。可以直接使用bean标签中的name属性：

```xml
<bean id="testBean" name="testBean,testBean2" class="com.TestBean"/>
```

同时Spring还提供了另一种声明别名的方式：

```xml
<bean id="testBean" class="com.TestBean"/>
<alias nam"testBean" alias="testBean,testBean2"/>
```

考虑一个更为具体的例子：组件A在XML配置文件中定义了一个名为componentA的DataSource类型的bean，但组件B却想在其XML文件中以componentB命名来引用此bean。而且在主程序MyApp的XML配置文件中，希望以myApp的名字来引用此bean。最后容器加载3个XML文件来生成最终的ApplicationContext。在此情形下，可通过在配置文件中添加下列alias元素来实现：

```xml
<alias nam"componentA" alias="componentB"/>
<alias nam"componentA" alias="myApp"/>
```

这样一来，每个组件及主程序就可以通过唯一名字来引用同一个数据源而互不干扰。

## 3.3 import标签的解析

## 3.4 嵌入式beans标签的解析

# 第4章 自定义标签的解析





























