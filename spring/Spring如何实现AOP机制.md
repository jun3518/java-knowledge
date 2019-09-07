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

### Joinpoint

Joinpoint（连接点）：连接点是指那些被拦截到的点。在spring中,这些点指的是方法，因为spring只支持方法类型的连接点。

### Pointcut

Pointcut（切点）：切点是指我们要对哪些Joinpoint进行拦截的定义。

### Advice

Advice（通知/增强）：通知是指拦截到Joinpoint之后所要做的事情就是通知。通知分为前置通知、后置通知、异常通知、最终通知、环绕通知。

### Target

Target（目标对象）：要代理的目标对象。

### Weaving

Weaving（织入）：是指把增强应用到目标对象来创建新的代理对象的过程。

### Proxy

Proxy（代理）:一个类被AOP织入增强后，就产生一个结果代理类。

### Aspect
Aspect（切面）：切面是切入点和通知的结合。

### Advisor

Advisor（通知器）：通知器是一个切点和一个通知结合。通过Advisor，把Advice和Pointcut结合起来，这个结合为是IoC容器的借基础设施。

## Spring的AOP代理种类

### JDK动态代理

JDK动态代理是对实现了接口的类生成代理。

### CGLIB代理

CGLIB是对类生成新的子类，从而进行代理。CGLIB不能对final类和方法进行代理。

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

#### 使用JdkDynamicAopProxy创建代理

```java
@Override
public Object getProxy(ClassLoader classLoader) {
    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(
        this.advised, true);
    findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```

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
            // There is only getDecoratedClass() declared -> dispatch to proxy config.
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
        }
    } else if (advisor instanceof IntroductionAdvisor) {
        Interceptor[] interceptors = registry.getInterceptors(advisor);
        interceptorList.addAll(Arrays.asList(interceptors));
    }
    else {
        Interceptor[] interceptors = registry.getInterceptors(advisor);
        interceptorList.addAll(Arrays.asList(interceptors));
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

Spring对拦截器链的调用都是在ReflectiveMethodInvocation中通过proceed方法实现的。在proceed方法中，会逐个运行拦截器的拦截方法。在运行拦截器的拦截方法之前，需要对代理方法完成一个匹配判断，通过这个匹配判断来决定拦截器是否满足切面增强的额要求。途观现在已经运行到拦截器链的末尾，那么就会直接调用目标对象的实现方法；否则沿着拦截器链继续进行，得到下一个拦截器，通过这个拦截器进行matches判断，判断是否适用于横切增强的场景，如果是，从拦截器中得到通知器，并启动通知器的invoke方法进行切面增强。在这个过程结束后，会迭代调用proceed方法，直到拦截器链中的拦截器都完成以上的拦截过程为止。

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

对于MethodInterceptor方法拦截器，是Interceptor的一个重要子类（也是Advice的子类），主要方法：invoke。入参为：MethodInvocation。

对于@Before、@After、@AfterReturning、@AfterThrowing、Around，会分别构造AspectJMethodBeforeAdvice、AspectJAfterAdvice、AspectJAfterReturningAdvice、AspectJAfterThrowingAdvice、AspectJAroundAdvice。





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



































































































