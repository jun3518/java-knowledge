## 什么是循环依赖？

循环依赖是指：A在初始化时依赖于B，B在初始化时依赖于C，C在初始化时依赖于A，即：A->B，B->C，C->A。如果容器不处理循环依赖的话，容器会无限执行上面的流程，直到内存溢出，程序崩溃。

## Spring有几种循环依赖？

Spring有3种循环依赖

在Spring中相应的配置如下：

```java
// A类
package spring.beans;
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class A {
	private B b;
}

// B类
package spring.beans;
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class B {
	private C c;
}

// C类
package spring.beans;
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class C {
	private A a;
}
```

Spring的XML配置：

```xml
<bean id="a" class="spring.beans.A">
    <constructor-arg name="b" ref="b"/>
</bean>
<bean id="b" class="spring.beans.B">
    <constructor-arg name="c" ref="c"/>
</bean>
<bean id="c" class="spring.beans.C">
    <constructor-arg name="a" ref="a"/>
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
Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classA' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'classB' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classB' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'classC' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'classC' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'classA' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'classA': Requested bean is currently in creation: Is there an unresolvable circular reference?
```

从报错的信息中可以得知，程序发生了循环依赖导致程序终止。

## Spring是如何解决循环依赖的呢？

循环依赖分为3种：

（1）单例构造器方式注入循环依赖。

（2）单例Setter方式注入循环依赖。

（3）原型Setter方式注入循环依赖。

其中构造器注入循环依赖Spring无法解决。因为在创建ClassA类时，构造器需要ClassB类，那将去创建ClassB，在创建ClassB类时又发现需要ClassC类，则又去创建ClassC，最终在创建ClassC时发现又需要ClassA，从而形成一个环，容器会无限执行上面的流程，直到内存溢出，程序崩溃。

### Spring解决Setter注入循环依赖

