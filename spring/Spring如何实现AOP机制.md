## Spring开启AOP例子

### 定义切面

```java
@Aspect
public class AspectTest {

    @Pointcut("execution(* *.test(..))")
    public void test() {
    }

    @Before("test()")
    public void beforeTest() {
        System.out.println("beforeTest");
    }

    @After("test()")
    public void afterTest() {
        System.out.println("afterTest");
    }

    @Around("test()")
    public Object aroundTest(ProceedingJoinPoint point) throws Throwable {
        System.out.println("aroundBeforeTest");
        Object object = point.proceed();
        System.out.println("aroundAfterTest");
        return object;
    }

}
```

### 定义Bean

```java
public class TestBean {

    public void test(){
        System.out.println("test");
    }

}
```

### 定义配置类

```java
// @EnableAspectJAutoProxy为开启事务的注解
@EnableAspectJAutoProxy
@Configuration
public class BeanConfig {

    @Bean
    public TestBean testBean() {
        return new TestBean();
    }

    @Bean
    public AspectTest aspectTest() {
        return new AspectTest();
    }

}
```

### 运行

```java
public static void main(String[] args) {

    AnnotationConfigApplicationContext context = 
        		new AnnotationConfigApplicationContext(BeanConfig.class);

    TestBean testBean = context.getBean("testBean", TestBean.class);

    testBean.test();

    context.close();

}
/*
运行结果：
aroundBeforeTest
beforeTest
test
aroundAfterTest
afterTest
*/
```

## AOP相关概念

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
| Pointcut             | 切点。是指要对哪些Joinpoint进行拦截的定义。                  |
| Aspect               | 切面。切面是切入点和通知的结合。                             |
| Advisor              | SpringAOP中的核心类。组合了Advice。                          |
| PointcutAdvisor      | SpringAOP中Advisor的重要子类。组合了切点(Pointcut)和Advice。 |
| BeforeAdvice         | 前置通知类。直接继承了Advice接口。                           |
| MethodBeforeAdvice   | BeforeAdvice的子类。定义了方法before。执行前置通知。         |
| AfterAdvice          | 后置通知类。直接继承了Advice接口。                           |
| ThrowsAdvice         | 后置异常通知类。直接继承了AfterAdvice接口。                  |
| AfterReturningAdvice | 后置返回通知类。直接继承了AfterAdvice接口，定义了方法beforeafterReturning，执行后置后置通知。 |
| @Before              | 前置通知，相当于BeforeAdvice。                               |
| @AfterReturning      | 后置通知，相当于AfterReturningAdvice。                       |
| @Around              | 环绕通知，相当于MethodInterceptor。                          |
| @AfterThrowing       | 抛出通知，相当于ThrowAdvice。                                |
| @After               | 最终final通知，不管是否异常，该通知都会执行。                |

有了对上述概念的理解，才能进行Spring AOP源码的分析。

## Spring的AOP组件注册

pring对AOP的支持是可拔插。要使用AOP时需要引入相应的依赖外，要需要添加一些配置。

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

## AnnotationAwareAspectJAutoProxyCreator

查看AnnotationAwareAspectJAutoProxyCreator的集成关系图：

![](./images/AnnotationAwareAspectJAutoProxyCreator继承关系.png)

在AnnotationAwareAspectJAutoProxyCreator的父类AbstractAutoProxyCreator中实现了BeanPostProcessor接口。BeanPostProcessor接口是后置处理器，是在Spring实例化bean的时候，对bean进行增强。

```java
public interface BeanPostProcessor {

    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
   
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

了解Spring的bean初始化流程的应该知道，BeanPostProccessor执行的时期为bean实例化，但未完成初始化的时期：

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 执行BeanPostProcessor#postProcessBeforeInitialization
        wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
    }

    this.invokeInitMethods(beanName, wrappedBean, mbd);

    if (mbd == null || !mbd.isSynthetic()) {
        // 执行BeanPostProcessor#postProcessAfterInitialization
        wrappedBean = 
            this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```

所以想要了解Spring是如何完成AOP的功能，那么可以查看AbstractAutoProxyCreator中对BeanPostProccessor两个方法的实现：

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
		implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
	
	//...
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) {
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (!this.earlyProxyReferences.contains(cacheKey)) {
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}
	//...
}
```

从上面的代码来看，postProcessBeforeInitialization没有做任何逻辑，真正做逻辑操作的是postProcessAfterInitialization。

对于getCacheKey方法，逻辑比较简单，如果bean是FactoryBean类型，那么返回“&+beanName”，否则直接返回bean对应的Class对象。

主要来看wrapIfNecessary方法：

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    // 如果已经处理过，则忽略
    if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    // 如果已经被标记为不需要增强，则忽略（之前已经处理过）
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    // 判断bean是否符合增强的条件，不符合则忽略
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)){
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    // 进行增强的逻辑操作
    // 获取bean对应的Advisor
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(
        bean.getClass(), beanName, null);
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        Object proxy = createProxy(bean.getClass(), beanName, specificInterceptors, 
                                   new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

对于AbstractAutoProxyCreator的isInfrastructureClass方法、shouldSkip方法和getAdvicesAndAdvisorsForBean方法，AbstractAutoProxyCreator的子类都有相应的覆盖重写。这是明显的模板方法模式。

模板方法模式：定义一个操作中的算法骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重新定义该算法的某些特定步骤。

即在AbstractAutoProxyCreator#postProcessAfterInitialization方法中，可以根据定义的算法骨架，可以从全局的角度清晰地了解整个Spring对AOP的逻辑步骤，然后再对某个步骤进行深入分析。

wrapIfNecessary方法中的逻辑步骤如下：

（1）判断bean是否是需要增强，不符合条件的直接返回bean。

（2）获取符合bean的Advisor。

（3）根据Advisor和一些个性化配置（如是否强制使用CGLIB进行代理等），创建相应的代理。

### AnnotationAwareAspectJAutoProxyCreator判断bean是否需要增强

```java
// 判断bean是否需要增强
if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}

//-----------------------下面的方法都是判断bean是否需要增强的具体逻辑---------------------

// AnnotationAwareAspectJAutoProxyCreator#isInfrastructureClass
@Override
protected boolean isInfrastructureClass(Class<?> beanClass) {
    return (super.isInfrastructureClass(beanClass) 
            || this.aspectJAdvisorFactory.isAspect(beanClass));
}

// AbstractAutoProxyCreator#isInfrastructureClass
protected boolean isInfrastructureClass(Class<?> beanClass) {
    boolean retVal = Advice.class.isAssignableFrom(beanClass) ||
        Pointcut.class.isAssignableFrom(beanClass) ||
            Advisor.class.isAssignableFrom(beanClass) ||
                AopInfrastructureBean.class.isAssignableFrom(beanClass);
    return retVal;
}

// AbstractAspectJAdvisorFactory#isAspect
@Override
public boolean isAspect(Class<?> clazz) {
    return (AnnotationUtils.findAnnotation(clazz, Aspect.class) != null);
}

// AspectJAwareAdvisorAutoProxyCreator#shouldSkip
@Override
protected boolean shouldSkip(Class<?> beanClass, String beanName) {
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    for (Advisor advisor : candidateAdvisors) {
        if (advisor instanceof AspectJPointcutAdvisor) {
            if (((AbstractAspectJAdvice) advisor.getAdvice())
                .getAspectName().equals(beanName)) {
                return true;
            }
        }
    }
    return super.shouldSkip(beanClass, beanName);
}

// AbstractAutoProxyCreator#shouldSkip
protected boolean shouldSkip(Class<?> beanClass, String beanName) {
    return false;
}
```

从上面的代码中还是可以较清晰地看到：如果是Advice、Pointcut、AopInfrastructureBean或者有Aspect注解的bean，都是不需要增强的bean。不难理解，因为Advice、Pointcut、AopInfrastructureBean或者有Aspect注解的bean都是用于定义AOP相关的基础bean，不是真实有业务逻辑的。

### AnnotationAwareAspectJAutoProxyCreator获取符合bean的Advisor

在AbstractAutoProxyCreator#getAdvicesAndAdvisorsForBean方法定义了获取符合bean的Advisor，是一个抽象方法，具体的逻辑实现在AbstractAdvisorAutoProxyCreator#getAdvicesAndAdvisorsForBean中：

```java
// 获取符合条件的所有Advisor
@Override
protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, 
                                                TargetSource targetSource) {
    // // 寻找所有Advisor中适用于bean的Advisor
    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) {
        return DO_NOT_PROXY;
    }
    return advisors.toArray();
}

// AbstractAdvisorAutoProxyCreator#findEligibleAdvisors
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    // 获取所有的Advisor
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    // 寻找所有Advisor中适用于bean的Advisor
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(
        candidateAdvisors, beanClass, beanName);
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```

#### 获取所有的Advisor

对于获取所有的Advisor的findCandidateAdvisors方法，AnnotationAwareAspectJAutoProxyCreator重写了findCandidateAdvisors方法，在方法中仍会调用父类的findCandidateAdvisors方法，但是新增了对标记有@Aspect注解的切面，进行解析获取Advisor。

```java
//AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors
protected List<Advisor> findCandidateAdvisors() {
    // 获取Spring容器中Advisor类型的bean
    List<Advisor> advisors = super.findCandidateAdvisors();
    // 获取Aspect切面的Advisor
    advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    return advisors;
}
```

super.findCandidateAdvisors()方法是从Spring容器中获取所有Advisor类型的bean。

```java
// AbstractAdvisorAutoProxyCreator#findCandidateAdvisors
protected List<Advisor> findCandidateAdvisors() {
    return this.advisorRetrievalHelper.findAdvisorBeans();
}
//BeanFactoryAdvisorRetrievalHelper#findAdvisorBeans
public List<Advisor> findAdvisorBeans() {
    // ...
    advisorNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
        this.beanFactory, Advisor.class, true, false);
	// ...
    List<Advisor> advisors = new ArrayList<Advisor>();
    for (String name : advisorNames) {
        advisors.add(this.beanFactory.getBean(name, Advisor.class));
    }
    // ...
    return advisors;
}
```

this.aspectJAdvisorsBuilder.buildAspectJAdvisors()方法是获取标记有@Aspect注解的切面，进行解析获取Advisor。

```java
// BeanFactoryAspectJAdvisorsBuilder#buildAspectJAdvisors
// 这是简化后的代码
public List<Advisor> buildAspectJAdvisors() {
    List<String> aspectNames = this.aspectBeanNames;
    List<Advisor> advisors = new LinkedList<Advisor>();
    aspectNames = new LinkedList<String>();
    // 获取容器中的所有beanName
    String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
        this.beanFactory, Object.class, true, false);
    for (String beanName : beanNames) {
        Class<?> beanType = this.beanFactory.getType(beanName);
        // 如果bean类型是Aspect，则解析Aspect中的切面信息
        if (this.advisorFactory.isAspect(beanType)) {
            aspectNames.add(beanName);
            // 将Aspect类型的beanName封装到BeanFactoryAspectInstanceFactory中
            MetadataAwareAspectInstanceFactory factory =
                new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
            // 获取Aspect得切面信息，并将其封装成Advisor
            List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
            this.advisorsCache.put(beanName, classAdvisors);
            advisors.addAll(classAdvisors);
        }
    }
    this.aspectBeanNames = aspectNames;
    return advisors;
}
```

在this.advisorFactory.getAdvisors(factory)方法中，其实做了很多的操作：

```java
public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
    // aspectClass为传入BeanFactoryAspectInstanceFactory的Aspect类型的beanName对应Class
    Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
    validate(aspectClass);
    MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
        new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);

    List<Advisor> advisors = new ArrayList<Advisor>();

    for (Method method : getAdvisorMethods(aspectClass)) {
        Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, 
                                     advisors.size(), aspectName);
        if (advisor != null) {
            advisors.add(advisor);
        }
    }
    // ...
    return advisors;
}

// ReflectiveAspectJAdvisorFactory#getAdvisor
public Advisor getAdvisor(Method candidateAdviceMethod, 
                          MetadataAwareAspectInstanceFactory aspectInstanceFactory,
                          int declarationOrderInAspect, String aspectName) {

    AspectJExpressionPointcut expressionPointcut = getPointcut(candidateAdviceMethod, 
							aspectInstanceFactory.getAspectMetadata().getAspectClass());
    if (expressionPointcut == null) {
        return null;
    }
    return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, 
                              candidateAdviceMethod, this, aspectInstanceFactory, 	
                                           		declarationOrderInAspect, aspectName);
}

public InstantiationModelAwarePointcutAdvisorImpl(
    AspectJExpressionPointcut declaredPointcut, Method aspectJAdviceMethod, 	
    AspectJAdvisorFactory aspectJAdvisorFactory,
    MetadataAwareAspectInstanceFactory aspectInstanceFactory, 
    int declarationOrder, String aspectName) {

    this.declaredPointcut = declaredPointcut;
    this.declaringClass = aspectJAdviceMethod.getDeclaringClass();
    this.methodName = aspectJAdviceMethod.getName();
    this.parameterTypes = aspectJAdviceMethod.getParameterTypes();
    this.aspectJAdviceMethod = aspectJAdviceMethod;
    this.aspectJAdvisorFactory = aspectJAdvisorFactory;
    this.aspectInstanceFactory = aspectInstanceFactory;
    this.declarationOrder = declarationOrder;
    this.aspectName = aspectName;
    //...
    this.pointcut = this.declaredPointcut;
    this.lazy = false;
    this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
}

private Advice instantiateAdvice(AspectJExpressionPointcut pcut) {
    return this.aspectJAdvisorFactory.getAdvice(this.aspectJAdviceMethod, pcut,
                 this.aspectInstanceFactory, this.declarationOrder, this.aspectName);
}

public Advice getAdvice(Method candidateAdviceMethod, 
                        AspectJExpressionPointcut expressionPointcut,
                        MetadataAwareAspectInstanceFactory aspectInstanceFactory, 
                        int declarationOrder, String aspectName) {

    Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    validate(candidateAspectClass);

    AspectJAnnotation<?> aspectJAnnotation =
      AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
    
    AbstractAspectJAdvice springAdvice;

    switch (aspectJAnnotation.getAnnotationType()) {
        case AtPointcut:
            return null;
        case AtAround:
            springAdvice = new AspectJAroundAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtBefore:
            springAdvice = new AspectJMethodBeforeAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfter:
            springAdvice = new AspectJAfterAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfterReturning:
            springAdvice = new AspectJAfterReturningAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterReturning afterReturningAnnotation = 
                (AfterReturning) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterReturningAnnotation.returning())) {
                springAdvice.setReturningName(afterReturningAnnotation.returning());
            }
            break;
        case AtAfterThrowing:
            springAdvice = new AspectJAfterThrowingAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterThrowing afterThrowingAnnotation = 
                (AfterThrowing) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterThrowingAnnotation.throwing())) {
                springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
            }
            break;
        default:
            throw new UnsupportedOperationException("...");
    }

    springAdvice.setAspectName(aspectName);
    springAdvice.setDeclarationOrder(declarationOrder);
    String[] argNames = 
        this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
    if (argNames != null) {
        springAdvice.setArgumentNamesFromStringArray(argNames);
    }
    springAdvice.calculateArgumentBindings();
    return springAdvice;
}
```

从代码的长度可以看出，只是获取一个Aspect对应的Advisor经过了很复杂的逻辑，其解析细节为：Spring会对Aspect切面中定义的方法进行解析，如果方法有Before、Around、After、AfterReturning、AfterThrowing、Pointcut等注解，则获取这些注解的信息并封装成AspectJExpressionPointcut对象，然后再赋值给InstantiationModelAwarePointcutAdvisorImpl。其中在InstantiationModelAwarePointcutAdvisorImpl构造器中会调用InstantiationModelAwarePointcutAdvisorImpl#instantiateAdvice方法，对增强类型注解包装成不同类型的Advice实例：Before -> AspectJMethodBeforeAdvice，After -> AspectJAfterAdvice，AfterReturning -> AspectJAfterReturningAdvice，AtAfterThrowing -> AspectJAfterThrowingAdvice，Around -> AspectJAroundAdvice。

解析完成后，对Aspect切面中定义的增强，全部都会封装成Advisor类型的InstantiationModelAwarePointcutAdvisorImpl对象。这样Spring就完成了获取所有Advisor和Aspect切面的解析。

注：InstantiationModelAwarePointcutAdvisorImpl是Advisor接口的实现类，该类中包含了具体的Advice实例。

#### 寻找所有增强中适用于bean的增强

```java
protected List<Advisor> findAdvisorsThatCanApply(
    List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {

    ProxyCreationContext.setCurrentProxiedBeanName(beanName);
    try {
        return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
    }
    finally {
        ProxyCreationContext.setCurrentProxiedBeanName(null);
    }
}

public static List<Advisor> findAdvisorsThatCanApply(
    List<Advisor> candidateAdvisors, Class<?> clazz) {
    if (candidateAdvisors.isEmpty()) {
        return candidateAdvisors;
    }
    List<Advisor> eligibleAdvisors = new LinkedList<Advisor>();
    // 先处理引介类型的Advisor
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
            eligibleAdvisors.add(candidate);
        }
    }
    boolean hasIntroductions = !eligibleAdvisors.isEmpty();
    for (Advisor candidate : candidateAdvisors) {
        // 引介类型的Advisor已经处理过，忽略
        if (candidate instanceof IntroductionAdvisor) {
            continue;
        }
        if (canApply(candidate, clazz, hasIntroductions)) {
            eligibleAdvisors.add(candidate);
        }
    }
    return eligibleAdvisors;
}

public static boolean canApply(Advisor advisor, Class<?> targetClass, 
                               boolean hasIntroductions) {
    if (advisor instanceof IntroductionAdvisor) {
        return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
    } else if (advisor instanceof PointcutAdvisor) {
        // PointcutAdvisor是InstantiationModelAwarePointcutAdvisorImpl类型实例
        PointcutAdvisor pca = (PointcutAdvisor) advisor;
        // pca.getPointcut()返回的是AspectJExpressionPointcut，它是在ReflectiveAspectJAdvisorFactory#getAdvisor中实例化，并通过构造函数加入InstantiationModelAwarePointcutAdvisorImpl
        return canApply(pca.getPointcut(), targetClass, hasIntroductions);
    } else {
        return true;
    }
}

public static boolean canApply(Pointcut pc, Class<?> targetClass, 
                               boolean hasIntroductions) {
    if (!pc.getClassFilter().matches(targetClass)) {
        return false;
    }
    // pc为AspectJExpressionPointcut，pc.getMethodMatcher()返回的是this，即AspectJExpressionPointcut本身
    MethodMatcher methodMatcher = pc.getMethodMatcher();
    if (methodMatcher == MethodMatcher.TRUE) {
        return true;
    }

    IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
    if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
        introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
    }

    Set<Class<?>> classes = new LinkedHashSet<Class<?>>
        (ClassUtils.getAllInterfacesForClassAsSet(targetClass));
    classes.add(targetClass);
    for (Class<?> clazz : classes) {
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
        for (Method method : methods) {
            if ((introductionAwareMethodMatcher != null &&
                 introductionAwareMethodMatcher.matches(
                     method, targetClass, hasIntroductions)) ||
                methodMatcher.matches(method, targetClass)) {
                return true;
            }
        }
    }

    return false;
}
```

查找符合条件的增强逻辑看似很多，但是逻辑不复杂，大致如下：

对于寻找所有增强中适用于bean的增强，Spring对所有增强声明的切点，对bean进行正则表达式匹配，如果符合，就加入适用的增强集合中。这样就可以挑选出适用的所有增强器了。

### 根据增强，创建相应的代理

根据获取到的增强，对bean进行生成代理的操作。逻辑在AbstractAutoProxyCreator#createProxy方法中：

```java
protected Object createProxy(Class<?> beanClass, String beanName, 
                             Object[] specificInterceptors, TargetSource targetSource) {

    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass(
            (ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }
    // 创建代理工厂，代理工厂用于创建真正的代理
    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);

    // 对proxyFactory装配个性化设置，用于创建代理时使用哪种方式创建（CGLIB还是JDK）
    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }
    // 装配增强信息
    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);

    proxyFactory.setFrozen(this.freezeProxy);
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }
    // 创建代理	
    return proxyFactory.getProxy(getProxyClassLoader());
}
// ProxyFactory#getProxy(java.lang.ClassLoader)
public Object getProxy(ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader);
}
// ProxyCreatorSupport#createAopProxy
protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    return getAopProxyFactory().createAopProxy(this);
}
// ProxyCreatorSupport#getAopProxyFactory
public AopProxyFactory getAopProxyFactory() {
    // aopProxyFactory在ProxyCreatorSupport构造器中会进行实例化DefaultAopProxyFactory
    return this.aopProxyFactory;
}
// DefaultAopProxyFactory#createAopProxy
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    
    if (config.isOptimize() || config.isProxyTargetClass() 
        				|| hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("...");
        }
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new ObjenesisCglibAopProxy(config);
    } else {
        return new JdkDynamicAopProxy(config);
    }
}
```

在DefaultAopProxyFactory#createAopProxy方法中，对bean进行创建代理类。当碰到以下三种情况时会使用CGLIB：
（1）ProxyConfig的isOptimize方法为true，这表示让Spring自己去优化。可以通过 ProxyFactory 的 setOptimize(true) 方法让 ProxyFactory 启动优化代理方式，这样，针对接口的代理也会使用 CglibAopProxy。
（2）ProxyConfig的isProxyTargetClass方法为true，这表示配置了proxy-target-class=”true”。
（3）ProxyConfig满足hasNoUserSuppliedProxyInterfaces方法执行结果为true，这表示bean没有实现任何接口或者实现的接口是SpringProxy接口。

### JdkDynamicAopProxy创建代理执行过程

JdkDynamicAopProxy类实现了java.lang.reflect.InvocationHandler接口。即JDK动态代理的实现逻辑位于JdkDynamicAopProxy#invoke方法中：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    MethodInvocation invocation;
    Object oldProxy = null;
    boolean setProxyContext = false;

    TargetSource targetSource = this.advised.targetSource;
    Class<?> targetClass = null;
    Object target = null;

    try {
        if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
            // 如果目标对象没有实现Object类的基本方法：equals
            return equals(args[0]);
        } else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
            // 如果目标对象没有实现Object类的基本方法：hashCode
            return hashCode();
        } else if (method.getDeclaringClass() == DecoratingProxy.class) {
            return AopProxyUtils.ultimateTargetClass(this.advised);
        } else if (!this.advised.opaque && method.getDeclaringClass().isInterface() 
                   && method.getDeclaringClass().isAssignableFrom(Advised.class)) {
            // 根据代理对象的配置来调用服务
            return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
        }

        Object retVal;

        if (this.advised.exposeProxy) {
            // Make invocation available if necessary.
            oldProxy = AopContext.setCurrentProxy(proxy);
            setProxyContext = true;
        }

        // 得到目标对象
        target = targetSource.getTarget();
        if (target != null) {
            targetClass = target.getClass();
        }

        // 获取拦截器链
        List<Object> chain = this.advised.
            getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

        //如果没有拦截器链，则直接调用target的对应方法
        if (chain.isEmpty()) {
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
        } else {
            // 如果有拦截器链，那么需要调用拦截器之后才调用目标对象的相应方法。
            // 通过构造一个ReflectiveMethodInvocation来实现。
            invocation = new ReflectiveMethodInvocation(
                proxy, target, method, args, targetClass, chain);
            // 沿着拦截器链继续前进
            retVal = invocation.proceed();
        }

        // Massage return value if necessary.
        Class<?> returnType = method.getReturnType();
        if (retVal != null && retVal == target 
            && returnType != Object.class && returnType.isInstance(proxy) 
            && !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
            retVal = proxy;
        }
        else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
            throw new AopInvocationException("...");
        }
        return retVal;
    } finally {
        if (target != null && !targetSource.isStatic()) {
            targetSource.releaseTarget(target);
        }
        if (setProxyContext) {
            AopContext.setCurrentProxy(oldProxy);
        }
    }
}
```

在this.advised.getInterceptorsAndDynamicInterceptionAdvice方法中，会将Advisor转换成MethodInterceptor的操作：

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, 
                                                                Class<?> targetClass) {
    MethodCacheKey cacheKey = new MethodCacheKey(method);
    List<Object> cached = this.methodCache.get(cacheKey);
    if (cached == null) {
        cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
            this, method, targetClass);
        this.methodCache.put(cacheKey, cached);
    }
    return cached;
}
//DefaultAdvisorChainFactory#getInterceptorsAndDynamicInterceptionAdvice
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
    Advised config, Method method, Class<?> targetClass) {

    for (Advisor advisor : config.getAdvisors()) {
        if (advisor instanceof PointcutAdvisor) {
            MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
            interceptorList.addAll(Arrays.asList(interceptors));
        } else if (advisor instanceof IntroductionAdvisor) {
            Interceptor[] interceptors = registry.getInterceptors(advisor);
            interceptorList.addAll(Arrays.asList(interceptors));
        } else {
            Interceptor[] interceptors = registry.getInterceptors(advisor);
            interceptorList.addAll(Arrays.asList(interceptors));
        }
    }
    return interceptorList;
}

// DefaultAdvisorAdapterRegistry#getInterceptors
public MethodInterceptor[] getInterceptors(Advisor advisor) {
    List<MethodInterceptor> interceptors = new ArrayList<MethodInterceptor>(3);
    Advice advice = advisor.getAdvice();
    if (advice instanceof MethodInterceptor) {
        interceptors.add((MethodInterceptor) advice);
    }
    for (AdvisorAdapter adapter : this.adapters) {
        if (adapter.supportsAdvice(advice)) {
            interceptors.add(adapter.getInterceptor(advisor));
        }
    }
    if (interceptors.isEmpty()) {
        throw new UnknownAdviceTypeException(advisor.getAdvice());
    }
    return interceptors.toArray(new MethodInterceptor[interceptors.size()]);
}

// 适配器，针对不同的Advisor，将Advisor包装成不同的MethodInterceptor
public DefaultAdvisorAdapterRegistry() {
    registerAdvisorAdapter(new MethodBeforeAdviceAdapter());
    registerAdvisorAdapter(new AfterReturningAdviceAdapter());
    registerAdvisorAdapter(new ThrowsAdviceAdapter());
}
```

对于@Before、@After、@AfterReturning、@AfterThrowing、@Around，会分别构造AspectJMethodBeforeAdvice、AspectJAfterAdvice、AspectJAfterReturningAdvice、AspectJAfterThrowingAdvice、AspectJAroundAdvice。

AspectJAfterAdvice、AspectJAroundAdvice都实现了MethodInterceptor接口，而AspectJMethodBeforeAdvice、AspectJAfterReturningAdvice、AspectJAfterThrowingAdvice接口由于没有实现MethodInterceptor接口，所以需要经过适配器进行转换：AspectJMethodBeforeAdvice -> MethodBeforeAdviceInterceptor，AspectJAfterReturningAdvice -> AfterReturningAdviceInterceptor，AspectJAfterThrowingAdvice -> ThrowsAdviceInterceptor。

这样，所有的Advice都会转化为MethodInterceptor。

```java
//AspectJAfterAdvice
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        return mi.proceed();
    }
    finally {
        // 最终通知，无论如何都会执行
        invokeAdviceMethod(getJoinPointMatch(), null, null);
    }
}
// AspectJAroundAdvice
public Object invoke(MethodInvocation mi) throws Throwable {
    if (!(mi instanceof ProxyMethodInvocation)) {
        throw new IllegalStateException("...");
    }
    ProxyMethodInvocation pmi = (ProxyMethodInvocation) mi;
    ProceedingJoinPoint pjp = lazyGetProceedingJoinPoint(pmi);
    JoinPointMatch jpm = getJoinPointMatch(pmi);
    return invokeAdviceMethod(pjp, jpm, null, null);
}
// AspectJMethodBeforeAdvice -> MethodBeforeAdviceInterceptor
public Object invoke(MethodInvocation mi) throws Throwable {
    // 前置通知，先执行通知
    this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
    return mi.proceed();
}
// AspectJAfterReturningAdvice -> AfterReturningAdviceInterceptor
public Object invoke(MethodInvocation mi) throws Throwable {
    Object retVal = mi.proceed();
    // 后置通知，后面再执行通知
    this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
    return retVal;
}
// AspectJAfterThrowingAdvice -> ThrowsAdviceInterceptor
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        return mi.proceed();
    }
    catch (Throwable ex) {
        // 异常通知，发生异常时才执行通知
        Method handlerMethod = getExceptionHandler(ex);
        if (handlerMethod != null) {
            invokeHandlerMethod(mi, ex, handlerMethod);
        }
        throw ex;
    }
}
```

Spring对拦截器链的调用都是在ReflectiveMethodInvocation中通过proceed方法实现的。在proceed方法中，会逐个运行拦截器的拦截方法。如果Interceptor是InterceptorAndDynamicMethodMatcher类型，那么在运行拦截器的拦截方法之前，需要对代理方法完成一个匹配判断，通过这个匹配判断来决定拦截器是否满足切面增强的额要求。假设现在已经运行到拦截器链的末尾，那么就会直接调用目标对象的实现方法；否则沿着拦截器链继续进行，得到下一个拦截器，通过这个拦截器进行matches判断，判断是否适用于横切增强的场景，如果是，从拦截器中得到通知器，并启动通知器的invoke方法进行切面增强。在这个过程结束后，会迭代调用proceed方法，直到拦截器链中的拦截器都完成以上的拦截过程为止。

```java
public Object proceed() throws Throwable {
    //	private int currentInterceptorIndex = -1;
    // 如果拦截器链中的拦截器迭代调用完毕，这里开始调用target的函数，这个函数式通过反射机制完成的
    if (this.currentInterceptorIndex == 
        this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        return invokeJoinpoint();
    }
	// 这里沿着定义好的interceptorOrInterceptionAdvice链进行处理
    Object interceptorOrInterceptionAdvice =
        this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
    if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
        // 对拦截器进行动态匹配判断
        InterceptorAndDynamicMethodMatcher dm =
            (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
        if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
            return dm.interceptor.invoke(this);
        } else {
            // 动态匹配失败，则proceed会被递归调用，直到所有的拦截器都被运行过为止
            return proceed();
        }
    } else {
        // 如果是个interceptor，直接调用这个interceptor对应的方法
        return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
    }
}
```

对于MethodInterceptor方法拦截器，是Interceptor的一个重要子类（也是Advice的子类），主要方法：invoke。入参为：MethodInvocation。代码中传入this，则代表传入的是当前对象。





































































































