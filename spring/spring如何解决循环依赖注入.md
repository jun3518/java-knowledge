## 什么是循环依赖？

循环依赖是指：A在初始化时依赖于B，B在初始化时依赖于C，C在初始化时依赖于A，即：A->B，B->C，C->A。如果容器不处理循环依赖的话，容器会无限执行上面的流程，直到内存溢出，程序崩溃。

## Spring有几种循环依赖？

循环依赖分为3种：

（1）单例构造器方式注入循环依赖。

（2）单例Setter方式注入循环依赖。

（3）原型Setter方式注入循环依赖。

## 单例构造器方式注入循环依赖

### 单例构造器方式注入循环依赖示例

在Spring中相应的配置如下：

```java
// ClassA类
package beans;
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class ClassA {
	private ClassB classB;
}

// ClassB类
package beans;
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class ClassB {
	private ClassC classC;
}

// ClassC类
package beans;
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class ClassC {
	private ClassA classA;
}
```

Spring的XML配置：

```xml
<bean id="classA" class="beans.ClassA">
    <constructor-arg name="classB" ref="classB"/>
</bean>
<bean id="classB" class="beans.ClassB">
    <constructor-arg name="classC" ref="classC"/>
</bean>
<bean id="classC" class="beans.ClassC">
    <constructor-arg name="classA" ref="classA"/>
</bean>
```

程序启动：

```java
public static void main(String[] args) {
		ClassPathXmlApplicationContext context = 
            new ClassPathXmlApplicationContext("beans.xml");
        ClassA userBean = context.getBean("classA", ClassA.class);
        context.close();
    }
```

程序启动时报错：

```java
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classB' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'classC' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classC' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'classA' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'classA': Requested bean is currently in creation: Is there an unresolvable circular reference?
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:359)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:108)
	at org.springframework.beans.factory.support.ConstructorResolver.resolveConstructorArguments(ConstructorResolver.java:648)
	at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:145)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1201)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1103)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:312)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:308)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:351)
	... 17 more
```

从报错的信息中可以得知，程序发生了循环依赖导致程序终止。

跟踪异常栈：BeanDefinitionValueResolver.java:108：

```java
// BeanDefinitionValueResolver.resolveValueIfNecessary(Object, Object)
public Object resolveValueIfNecessary(Object argName, Object value) {
    if (value instanceof RuntimeBeanReference) {
        RuntimeBeanReference ref = (RuntimeBeanReference) value;
        return resolveReference(argName, ref);
    }
}
private Object resolveReference(Object argName, RuntimeBeanReference ref) {
    try {
        String refName = ref.getBeanName();
        refName = String.valueOf(doEvaluate(refName));
        if (ref.isToParent()) {
            if (this.beanFactory.getParentBeanFactory() == null) {
                throw new BeanCreationException("...");
            }
            return this.beanFactory.getParentBeanFactory().getBean(refName);
        } else {
            // 根据beanName获取Bean实例，其中refName是构造函数中的形参名称
            Object bean = this.beanFactory.getBean(refName);
            this.beanFactory.registerDependentBean(refName, this.beanName);
            return bean;
        }
    }
    catch (BeansException ex) {
        throw new BeanCreationException("...");
    }
}
```

查看调用栈：

![](./images/循环依赖-构造函数注入调用栈.jpg)

从上图可以看出，真正创建单例bean的逻辑是在AbstractAutowireCapableBeanFactory#doCreateBean中：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
    throws BeanCreationException {

    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = (instanceWrapper != null ? 
                         instanceWrapper.getWrappedInstance() : null);
    Class<?> beanType = (instanceWrapper != null ? 
                         instanceWrapper.getWrappedClass() : null);
    // ...
}
```



## Spring是如何解决循环依赖的呢？

循环依赖分为3种：

（1）单例构造器方式注入循环依赖。

（2）单例Setter方式注入循环依赖。

（3）原型Setter方式注入循环依赖。

其中构造器注入循环依赖Spring无法解决。因为在创建ClassA类时，构造器需要ClassB类，那将去创建ClassB，在创建ClassB类时又发现需要ClassC类，则又去创建ClassC，最终在创建ClassC时发现又需要ClassA，从而形成一个环，容器会无限执行上面的流程，直到内存溢出，程序崩溃。

### Spring解决Setter注入循环依赖

