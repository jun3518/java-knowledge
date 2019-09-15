## Bean的生命周期

Java中，一个Bean的生命周期大致分为3个阶段：

（1）创建。即通过构造函数或者反射方式创建的bean对象。

（2）初始化。创建对象之后，bean可能需要一些初始化操作，如设置一些属性等等。

（4）销毁。bean对象完成功能操作后，没有存在的必要了，就需要对其进行销毁。在JVM中，bean对象的销毁交由虚拟机去完成。

## Spring中对Bean的生命周期的管理

（1）InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation方法

（2）

### Spring对单例Bean的生命周期的管理

```java
@Data
@AllArgsConstructor
public class Dog {
    private String name;	
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person implements BeanFactoryAware, InitializingBean, DisposableBean {

    private String username;

    private String password;

    private Dog dog;

    public void setDog(Dog dog) {
        System.out.println("Person...setDog()");
        this.dog = dog;
    }

    public void initMethod() {
        System.out.println("Person...initMethod()");
    }

    public void destroyMethod() {
        System.out.println("Person...destroyMethod()");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Person...destroy()");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Person...afterPropertiesSet()");

    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("Person...postConstruct()");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("Person...preDestroy()");
    }

    public Person(String username, String password) {
        System.out.println("Person...Person(username, password)");
        this.username = username;
        this.password = password;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("Person...setBeanFactory()");

    }
}

@Component
public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInitialization()");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "...MyInstantiationAwareBeanPostProcessor...postProcessAfterInitialization()");
        return bean;
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        System.out.println(beanName + "...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInstantiation()");
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + 
 "...MyInstantiationAwareBeanPostProcessor...postProcessAfterInstantiation()");
        return false;
    }

    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, 
          PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
        System.out.println(beanName + 
				"MyInstantiationAwareBeanPostProcessor...postProcessPropertyValues()");
        return pvs;
    }

}

@ComponentScan("com.spring1")
@Configuration
public class BeanConfig {

    @Bean(initMethod="initMethod", destroyMethod="destroyMethod")
    public Person person() {
        Person person = new Person("zhangsan", "123456");
        person.setDog(dog());
        return person;
    }

    @Bean
    public Dog dog() {
        return new Dog("petter");
    }
}
```

打印结果：

```java
person...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInstantiation()
Person...Person(username, password)
dog...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInstantiation()
dog...MyInstantiationAwareBeanPostProcessor...postProcessAfterInstantiation()
dog...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInitialization()
dog...MyInstantiationAwareBeanPostProcessor...postProcessAfterInitialization()
Person...setDog()
person...MyInstantiationAwareBeanPostProcessor...postProcessAfterInstantiation()
Person...setBeanFactory()
person...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInitialization()
Person...postConstruct()
Person...afterPropertiesSet()
Person...initMethod()
person...MyInstantiationAwareBeanPostProcessor...postProcessAfterInitialization()
    九月 14, 2019 9:17:51 下午 org.springframework.context.annotation.AnnotationConfigApplicationContext doClose
信息: Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@7e6cbb7a: startup date [Sat Sep 14 21:17:50 CST 2019]; root of context hierarchy
Person...preDestroy()
Person...destroy()
Person...destroyMethod()
```

Spring在bean的创建和销毁的过程中，提供了很多对bean的扩展点。

Spring通过getBean()方法进行实例化bean实例的时候，做了以下逻辑：

（1）InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation(Class<?>, String)。

（2）实例化bean依赖的其他bean。

（3）执行bean的setter方法。

（4）执行InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation(Object, String)。

（5）执行set*Aware方法。

（6）执行BeanPostProcessor.postProcessBeforeInitialization(Object, String)方法。

（7）执行@PostConstruct标注的方法。

（8）执行InitializingBean.afterPropertiesSet()方法。

（9）执行@Bean(initMethod="initMethod")的initMethod指定的方法。

（10）执行BeanPostProcessor.postProcessAfterInitialization(Object, String)方法。

（11）执行@PreDestroy标注的方法。

（12）执行DisposableBean.destroy()方法。

（13）执行@Bean(destroyMethod="destroyMethod")的destroyMethod指定的方法。

### Spring对prototype Bean的生命周期的管理

```java
@Configuration
public class BeanConfig {
	@Scope("prototype")
	@Bean(initMethod="initMethod", destroyMethod="destroyMethod")
	public Person person() {
		Person person = new Person("zhangsan", "123456");
		person.setDog(dog());
		return person;
	}
	// ...
}
```

打印结果：

```java
dog...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInstantiation()
dog...MyInstantiationAwareBeanPostProcessor...postProcessAfterInstantiation()
dog...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInitialization()
dog...MyInstantiationAwareBeanPostProcessor...postProcessAfterInitialization()
person...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInstantiation()
Person...Person(username, password)
Person...setDog()
person...MyInstantiationAwareBeanPostProcessor...postProcessAfterInstantiation()
Person...setBeanFactory()
person...MyInstantiationAwareBeanPostProcessor...postProcessBeforeInitialization()
Person...postConstruct()
Person...afterPropertiesSet()
Person...initMethod()
person...MyInstantiationAwareBeanPostProcessor...postProcessAfterInitialization()
```

从打印结果来看，prototype Bean的生命周期执行了上面的（1）-（10）的步骤，关闭的相关方法没有执行。