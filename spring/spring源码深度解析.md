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

### 2.5.1 配置文件封装

