## Spring的AOP组件注册

### 通过配置文件注册

Spring通过在配置文件中加入元素开启注解切面功能：

```xml
<!-- 开启基于注解版的切面功能 -->
<aop:aspectj-autoproxy />
```

其中\<aop:aspectj-autoproxy />是自定义标签元素，自定义标签元素通过Spring提供的org.springframework.beans.factory.xml.NamespaceHandler接口来完成自定义标签的解析。

查看NamespaceHandler实现类，发现和AOP有关的未org.springframework.aop.config.AopNamespaceHandler。对于NamespaceHandler接口，程序执行时首先会执行NamespaceHandler#init，跟踪AopNamespaceHandler#init方法：

```java
public void init() {
    registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
    registerBeanDefinitionParser("aspectj-autoproxy", 
                                 new AspectJAutoProxyBeanDefinitionParser());
    registerBeanDefinitionDecorator("scoped-proxy", 
                                    new ScopedProxyBeanDefinitionDecorator());
    registerBeanDefinitionParser("spring-configured", 
                                 new SpringConfiguredBeanDefinitionParser());
}
```

从上面的代码中可以看出：对于“aspectj-autoproxy”自定义标签元素，会通过AspectJAutoProxyBeanDefinitionParser来完成解析：

```java
// Spring在解析自定义元素“aspectj-autoproxy”的时候，会调用AspectJAutoProxyBeanDefinitionParser#parse来完成解析
public BeanDefinition parse(Element element, ParserContext parserContext) {
    AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(
        parserContext, element);
    extendBeanDefinition(element, parserContext);
    return null;
}

public static void registerAspectJAnnotationAutoProxyCreatorIfNecessary(
    ParserContext parserContext, Element sourceElement) {
    BeanDefinition beanDefinition = 
        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(
        parserContext.getRegistry(), parserContext.extractSource(sourceElement));
    useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
    registerComponentIfNecessary(beanDefinition, parserContext);
}

public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(
    BeanDefinitionRegistry registry, Object source) {
    return registerOrEscalateApcAsRequired(
        AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}
//AUTO_PROXY_CREATOR_BEAN_NAME="org.springframework.aop.config.internalAutoProxyCreator";
private static BeanDefinition registerOrEscalateApcAsRequired(
    Class<?> cls, BeanDefinitionRegistry registry, Object source) {
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

从上面的代码来看，Spring对于自定义标签\<aop:aspectj-autoproxy />的解析逻辑是在Spring中添加一个beanName为“org.springframework.aop.config.internalAutoProxyCreator”，类为org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator的BeanDefinition组件，且这个BeanDefinition的“order”优先级是最高的。





