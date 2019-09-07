## AOP联盟中定义的类

| 类名              | 作用概述                                                     |
| ----------------- | ------------------------------------------------------------ |
| Advice            | AOP联盟定义的一个标识接口。通知和Interceptor顶级类。即各种通知类型都要实现这个接口。 |
| Interceptor       | AOP联盟中进行方法拦截的一个标识接口。是Advice的子类。        |
| MethodInterceptor | 方法拦截器。是Interceptor的一个重要子类。主要方法：invoke。入参为：MethodInvocation |
| Joinpoint         | AOP联盟中的连接点类。主要的方法是：proceed()执行下一个拦截器。getThis()获取目标对象。 |
| Invocation        | AOP拦截的执行类。是Joinpoint的子类。主要方法：getArguments()获取参数。 |
| MethodInvocation  | Invocation的一个重要实现类。真正执行AOP方法的拦截。主要方法：getMethod()目标方法。 |

![](https://img-blog.csdn.net/20180318112954851?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3prbnh4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



## SpringAOP中定义的类

| 类名                                                         | 作用概述                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Advisor                                                      | SpringAOP中的核心类。组合了Advice。                          |
| PointcutAdvisor                                              | SpringAOP中Advisor的重要子类。组合了切点(Pointcut)和Advice。 |
| InstantiationModelAwarePointcutAdvisorImpl                   | PointcutAdvisor的一个重要实现子类。                          |
| DefaultPointcutAdvisor                                       | PointcutAdvisor的另一个重要实现子类。可以将Advice包装为Advisor。在SpringAOP中是以Advisor为主线。向Advice靠拢。 |
| Pointcut                                                     | SpringAOP中切点的顶级抽象类。                                |
| TruePointcut                                                 | Pointcut的一个重要实现类。在DefaultPointcutAdvisor中使用的是TruePointcut。在进行切点匹配的时候永远返回true |
| AspectJExpressionPointcut                                    | Pointcut的一个重要实现类。AspectJ语法切点类。同时实现了MethodMatcher，AspectJ语法切点的匹配在这个类中完成。 |
| AnnotationMatchingPointcut                                   | Pointcut的一个重要实现类。注解语法的切点类。                 |
| JdkRegexpMethodPointcut                                      | Pointcut的一个重要实现类。正则语法的切点类。                 |
| MethodMatcher                                                | 切点匹配连接点的地方。即类中的某个方法和我们定义的切点表达式是否匹配、能不能被AOP拦截。 |
| TrueMethodMatcher                                            | 用于返回true。                                               |
| AnnotationMethodMatcher                                      | 带有注解的方法的匹配器。                                     |
| JdkRegexpMethodPointcut                                      | 正则表达式                                                   |
| Advised                                                      | SpringAOP中的又一个核心类。它组合了Advisor和TargetSource即目标对象。 |
| ProxyConfig                                                  | SpringAOP中的一个核心类。在Advised中定义了一系列的配置接口，像：是否暴露对象、是否强制使用CGlib等。ProxyConfig是对这些接口的实现，但是ProxyConfig却不是Advised的实现类。 |
| AdvisedSupport                                               | Advised的一个实现类。SpringAOP中的一个核心类。继承了ProxyConfig实现了Advised。 |
| ProxyCreatorSupport                                          | AdvisedSupport的子类。引用了AopProxyFactory用来创建代理对象。 |
| ProxyFactory                                                 | ProxyCreatorSupport的子类。用来创建代理对象。在SpringAOP中用的最多。 |
| ProxyFactoryBean                                             | ProxyCreatorSupport的子类。用来创建代理对象。它实现了BeanFactoryAware、FactoryBean接口。 |
| AspectJProxyFactory                                          | ProxyCreatorSupport的子类。用来创建代理对象。使用AspectJ语法。 |
| ProxyFactory、ProxyFactoryBean、AspectJProxyFactory这三个类的使用场景各不相同。 | 但都是生成Advisor和TargetSource、代理对象的关系。            |
| AbstractAutoProxyCreator                                     | ProxyProcessorSupport的重要子类。SpringAOP中的核心类。实现了SmartInstantiationAwareBeanPostProcessor、BeanFactoryAware接口。自动创建代理对象的类。我们在使用AOP的时候基本上都是用的这个类来进程Bean的拦截，创建代理对象。 |
| AbstractAdvisorAutoProxyCreator                              | AbstractAutoProxyCreator的子类。SpringAOP中的核心类。用来创建Advisor和代理对象。 |
| AspectJAwareAdvisorAutoProxyCreator                          | AbstractAdvisorAutoProxyCreator的子类。使用AspectJ语法创建Advisor和代理对象。 |
| AnnotationAwareAspectJAutoProxyCreator                       | AspectJAwareAdvisorAutoProxyCreator的子类。使用AspectJ语法创建Advisor和代理对象的类。<aop:aspectj-autoproxy />标签默认注入到SpringAOP中的BeanDefinition。 |
| TargetSource                                                 | 持有目标对象的接口。                                         |
| SingletonTargetSource                                        | TargetSource的子类。适用于单例目标对象。                     |
| AopProxy                                                     | 生成AOP代理对象的类。                                        |
| JdkDynamicAopProxy                                           | AopProxy的子类。使用JDK的方式创建代理对象。它持有Advised对象。 |
| CglibAopProxy                                                | AopProxy的子类。使用Cglib的方法创建代理对象。它持有Advised对象。 |
| AopProxyFactory                                              | 创建AOP代理对象的工厂类。选择使用JDK还是Cglib的方式来创建代理对象。 |
| DefaultAopProxyFactory                                       | AopProxyFactory的子类，也是SpringAOP中唯一默认的实现类。     |
| AdvisorChainFactory                                          | 获取Advisor链的接口。                                        |
| DefaultAdvisorChainFactory                                   | AdvisorChainFactory的实现类。也是SpringAOP中唯一默认的实现类。 |
| AdvisorAdapterRegistry                                       | Advisor适配注册器类。用来将Advice适配为Advisor。将Advisor适配为MethodInterceptor。 |
| DefaultAdvisorAdapterRegistry                                | AdvisorAdapterRegistry的实现类。也是SpringAOP中唯一默认的实现类。持有MethodBeforeAdviceAdapter、AfterReturningAdviceAdapter、ThrowsAdviceAdapter实例。 |
| BeforeAdvice                                                 | 前置通知类。直接继承了Advice接口。                           |
| MethodBeforeAdvice                                           | BeforeAdvice的子类。定义了方法before。执行前置通知。         |
| MethodBeforeAdviceInterceptor                                | MethodBefore前置通知Interceptor。实现了MethodInterceptor接口。持有MethodBefore对象。 |
| AfterAdvice                                                  | 后置通知类。直接继承了Advice接口。                           |
| ThrowsAdvice                                                 | 后置异常通知类。直接继承了AfterAdvice接口。                  |
| AfterReturningAdvice                                         | 后置返回通知类。直接继承了AfterAdvice接口。                  |
| AfterReturningAdviceInterceptor                              | 后置返回通知Interceptor。实现了MethodInterceptor和AfterAdvice接口。持有AfterReturningAdvice实例。 |
| ThrowsAdviceInterceptor | 后置异常通知Interceptor。实现了MethodInterceptor和AfterAdvice接口。要求方法名为：afterThrowing |
| AdvisorAdapter | Advisor适配器。判断此接口的是不是能支持对应的Advice。五种通知类型，只有三种通知类型适配器。 |
| MethodBeforeAdviceAdapter | 前置通知的适配器。支持前置通知类。有一个getInterceptor方法：将Advisor适配为MethodInterceptor。Advisor持有Advice类型的实例，获取MethodBeforeAdvice，将MethodBeforeAdvice适配为MethodBeforeAdviceInterceptor。AOP的拦截过程通过MethodInterceptor来完成。 |
| AfterReturningAdviceAdapter | 后置返回通知的适配器。支持后置返回通知类。有一个getInterceptor方法：将Advisor适配为MethodInterceptor。Advisor持有Advice类型的实例，获取AfterReturningAdvice，将AfterReturningAdvice适配为AfterReturningAdviceInterceptor。AOP的拦截过程通过MethodInterceptor来完成。 |
| ThrowsAdviceAdapter | 后置异常通知的适配器。支持后置异常通知类。有一个getInterceptor方法：将Advisor适配为MethodInterceptor。Advisor持有Advice类型的实例，获取ThrowsAdvice，将ThrowsAdvice适配为ThrowsAdviceInterceptor。AOP的拦截过程通过MethodInterceptor来完成。 |
| AspectJMethodBeforeAdvice | 使用AspectJ Before注解的前置通知类型。实现了MethodBeforeAdvice继承了AbstractAspectJAdvice。 |
| AspectJAfterAdvice | 使用AspectJ After注解的后置通知类型。实现了MethodInterceptor、AfterAdvice接口。继承了AbstractAspectJAdvice。 |
| AspectJAfterReturningAdvice | 使用AspectJ AfterReturning注解的后置通知类型。实现了AfterReturningAdvice、AfterAdvice接口。继承了AbstractAspectJAdvice。 |
| AspectJAroundAdvice | 使用AspectJ Around注解的后置通知类型。实现了MethodInterceptor接口。继承了AbstractAspectJAdvice。 |
| AspectJAfterThrowingAdvice | 使用AspectJ Around注解的后置通知类型。实现了MethodInterceptor、AfterAdvice接口。继承了AbstractAspectJAdvice。 |
| AspectJAdvisorFactory | 使用AspectJ注解 生成Advisor工厂类。 |
| AbstractAspectJAdvisorFactory | AspectJAdvisorFactory的子类。使用AspectJ注解 生成Advisor的工厂类。 |
| ReflectiveAspectJAdvisorFactory | AbstractAspectJAdvisorFactory的子类。使用AspectJ注解 生成Advisor的具体实现类。 |
| AspectMetadata | 使用AspectJ Aspect注解的切面元数据类。 |
| MetadataAwareAspectInstanceFactory | AspectInstanceFactory的子类。含有Aspect注解元数据 Aspect切面实例工厂类。 |
| ProxyMethodInvocation | 含有代理对象的。MethodInvocation的子类。 |
| ReflectiveMethodInvocation | ProxyMethodInvocation的子类。AOP拦截的执行入口类。 |
| CglibMethodInvocation | ReflectiveMethodInvocation的子类。对Cglib反射调用目标方法进行了一点改进。 |















