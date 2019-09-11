## AOP概述

AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

### AOP联盟中定义的类

| 类名              | 作用概述                                                     |
| ----------------- | ------------------------------------------------------------ |
| Advice            | AOP联盟定义的一个标识接口。通知和Interceptor顶级类。即各种通知类型都要实现这个接口。 |
| Interceptor       | AOP联盟中进行方法拦截的一个标识接口。是Advice的子类。        |
| MethodInterceptor | 方法拦截器。是Interceptor的一个重要子类。主要方法：invoke。入参为：MethodInvocation |
| Joinpoint         | AOP联盟中的连接点类。主要的方法是：proceed()执行下一个拦截器。getThis()获取目标对象。 |
| Invocation        | AOP拦截的执行类。是Joinpoint的子类。主要方法：getArguments()获取参数。 |
| MethodInvocation  | Invocation的一个重要实现类。真正执行AOP方法的拦截。主要方法：getMethod()目标方法。 |

![](https://img-blog.csdn.net/20180318112954851?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3prbnh4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



### SpringAOP中定义的类

| 类名                 | 作用概述                                                     |
| -------------------- | ------------------------------------------------------------ |
| Advisor              | SpringAOP中的核心类。组合了Advice。                          |
| PointcutAdvisor      | SpringAOP中Advisor的重要子类。组合了切点(Pointcut)和Advice。 |
| BeforeAdvice         | 前置通知类。直接继承了Advice接口。                           |
| MethodBeforeAdvice   | BeforeAdvice的子类。定义了方法before。执行前置通知。         |
| AfterAdvice          | 后置通知类。直接继承了Advice接口。                           |
| ThrowsAdvice         | 后置异常通知类。直接继承了AfterAdvice接口。                  |
| AfterReturningAdvice | 后置返回通知类。直接继承了AfterAdvice接口。                  |

有了对上述概念的理解，才能进行Spring AOP源码的分析。



## Spring的AOP组件注册

Spring对AOP的支持是可拔插。要使用AOP时需要引入相应的依赖外，要需要添加一些配置。

### 通过配置文件注册AOP组件

Spring通过在配置文件中加入元素开启注解切面功能：

```xml
<!-- 声明aop的命名空间 -->
<beans xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/aop 
                           http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">
    <!-- 开启基于注解版的切面功能 -->
    <aop:aspectj-autoproxy />
</beans>
```

对于自定义元素标签，Spring会通过接口org.springframework.beans.factory.xml.NamespaceHandler接口进行扩展。

对于<aop:aspectj-autoproxy />的自定义标签，Spring提供了org.springframework.aop.config.AopNamespaceHandler类对改标签进行解析操作。

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {
    
    @Override
    public void init() {
        registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
        registerBeanDefinitionParser("aspectj-autoproxy", 
                                     new AspectJAutoProxyBeanDefinitionParser());
        registerBeanDefinitionDecorator("scoped-proxy", 
                                        new ScopedProxyBeanDefinitionDecorator());
        registerBeanDefinitionParser("spring-configured", 
                                     new SpringConfiguredBeanDefinitionParser());
    }
}
```

可以发现，在init方法中注入了和<aop:aspectj-autoproxy />标签相关的AspectJAutoProxyBeanDefinitionParser解析器。

跟踪AspectJAutoProxyBeanDefinitionParser类：

```java
class AspectJAutoProxyBeanDefinitionParser implements BeanDefinitionParser {

    @Override
    public BeanDefinition parse(Element element, ParserContext parserContext) {
        // 注册一个AspectJAnnotationAutoProxyCreator bean
        AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(
            parserContext, element);
        extendBeanDefinition(element, parserContext);
        return null;
    }
    //...
}
```

继续跟踪AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary方法，在Spring中注册了一个beanName为org.springframework.aop.config.internalAutoProxyCreator到Spring容器中：

```java
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(
    BeanDefinitionRegistry registry, Object source) {
    return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class,
                                           registry, source);
}
private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
  //AUTO_PROXY_CREATOR_BEAN_NAME=org.springframework.aop.config.internalAutoProxyCreator
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition apcDefinition = 
            registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
            int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
            int requiredPriority = findPriorityForClass(cls);
            if (currentPriority < requiredPriority) {
                apcDefinition.setBeanClassName(cls.getName());
            }
        }
        return null;
    }

    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
    return beanDefinition;
}
```

### 通过注解的方式注册AOP组件

```java
// @EnableAspectJAutoProxy开启了注解切面功能
@EnableAspectJAutoProxy
@Configuration
public class BeanConfig {
    // ...
}
```

跟踪EnableAspectJAutoProxy注解的代码：

```java
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
    
	boolean proxyTargetClass() default false;
    
	boolean exposeProxy() default false;
}
```

从上面的代码中可知，EnableAspectJAutoProxy注解又通过Import注解导入了AspectJAutoProxyRegistrar类：

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(
        AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
        //...
    }
}
```

通过上面的代码可知，AspectJAutoProxyRegistrar实际上又调用了AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry)方法，即注册了AnnotationAwareAspectJAutoProxyCreator的bean到Spring容器中。