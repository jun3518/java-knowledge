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

## @EnableAspectJAutoProxy注解

查看@EnableAspectJAutoProxy注解源码：

```java
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
    //...
}

class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
    
    @Override
    public void registerBeanDefinitions(
        AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		// 向Spring中注册AnnotationAwareAspectJAutoProxyCreator的bean
        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
		//...
    }
    
}
```

根据上面的代码可知，@EnableAspectJAutoProxy会最终向Spring中注入AnnotationAwareAspectJAutoProxyCreator的bean。

## AnnotationAwareAspectJAutoProxyCreator

查看AnnotationAwareAspectJAutoProxyCreator的集成关系图：

![](E:\spring-aop\spring\images\AnnotationAwareAspectJAutoProxyCreator继承关系.png)

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

对于getCacheKey方法，逻辑比较简单，如果bean是FactoryBean类型，那么在&+beanName返回，否则直接返回bean对应的Class对象。

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
    // 获取bean对应的增强器
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

即在AbstractAutoProxyCreator#postProcessAfterInitialization方法中，可以根据定义的算法骨架，可以从全局的角度清晰地了解整个Spring对AOP的逻辑步骤，然后再对某个步骤进行深入分析，而不是一开始就深陷于Spring错综复杂的源码。

根据这个思路，Spring对AOP的实现逻辑步骤如下：

（1）判断bean是否是需要增强，不符合条件的进行记录。

（2）获取符合bean的增强器。

（3）根据增强器和一些个性化配置（如是否强制使用CGLIB进行代理等），创建相应的代理。

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

从上面的代码中还是可以较清晰地看到：如果是Advice、Pointcut、AopInfrastructureBean或者有Aspect注解的bean，都是不需要增强的bean。不难理解，因为Advice、Pointcut、AopInfrastructureBean或者有Aspect注解的bean都是用于定义AOP相关的bean，不是真实业务的bean。

### AnnotationAwareAspectJAutoProxyCreator获取符合bean的增强器

在AbstractAutoProxyCreator#getAdvicesAndAdvisorsForBean方法定义了获取符合bean的增强器，是一个抽象方法，具体的逻辑实现在AbstractAdvisorAutoProxyCreator#getAdvicesAndAdvisorsForBean中：

```java
// 获取符合条件的所有增强
@Override
protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, 
                                                TargetSource targetSource) {
    // // 寻找所有增强中适用于bean的增强
    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) {
        return DO_NOT_PROXY;
    }
    return advisors.toArray();
}

// AbstractAdvisorAutoProxyCreator#findEligibleAdvisors
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    // 获取所有的增强
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    // 寻找所有增强中适用于bean的增强
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(
        candidateAdvisors, beanClass, beanName);
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```

#### 获取所有的增强

对于获取所有的增强的findCandidateAdvisors方法，AnnotationAwareAspectJAutoProxyCreator重写了findCandidateAdvisors方法，在方法中仍会调用父类的findCandidateAdvisors方法，但是新增了对标记有@Aspect注解的切面，进行解析获取Advisor。

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

this.aspectJAdvisorsBuilder.buildAspectJAdvisors()方法是获取标记有@Aspect注解的切面，进行解析获取Advisor。其解析细节为：Spring会对Aspect切面中定义的方法进行解析，如果方法有Before、Around、After、AfterReturning、AfterThrowing、Pointcut等切点或增强的注解，则获取这些注解的表达式并封装成AspectJExpressionPointcut对象，然后再赋值给InstantiationModelAwarePointcutAdvisorImpl。其中在InstantiationModelAwarePointcutAdvisorImpl构造器中会调用InstantiationModelAwarePointcutAdvisorImpl#instantiateAdvice方法，对增强类型注解包装成不同类型的增强实例：

```java
switch (aspectJAnnotation.getAnnotationType()) {
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
    case AtAround:
        springAdvice = new AspectJAroundAdvice(
            candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
        break;
    case AtPointcut:
        return null;
    default:
        throw new UnsupportedOperationException("...");
}
```

注：InstantiationModelAwarePointcutAdvisorImpl是Advisor接口的实现类，该类中包含了具体的Advice实例。

解析完成后，对Aspect切面中定义的增强，全部都会封装成InstantiationModelAwarePointcutAdvisorImpl对象。这样Spring就完成了获取所有Advisor和Aspect切面的解析。

#### 寻找所有增强中适用于bean的增强

对于寻找所有增强中适用于bean的增强，Spring对所有增强声明的切点，对bean进行正则表达式匹配，如果符合，就加入适用的增强集合中。这样就可以挑选出适用的所有增强器了。这里只是对逻辑进行概述，具体的实现过于细节，不在这里展开。

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







































































































