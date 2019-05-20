* content
{:toc}

### 什么是AOP

在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

在Java中，利用代理机制实现AOP。



### AOP的术语

#### 连接点（Joinpoint）

连接点是指那些被拦截到的点。在spring中指的是方法，因为spring只支持方法类型的连接点。

#### 切入点（Pointcut）

切入点是指要对哪些连接点进行拦截的连接点。

#### 通知（Advice）

通知也成增强，是指拦截到连接点之后所要做的事情就是通知。通知分为前置通知、后置通知、异常通知、最终通知、环绕通知。

##### 前置通知

在目标方法执行前增强

##### 后置通知

在目标方法执行后增强

##### 环绕通知

在目标方法执行前后增强

##### 异常通知

在方法抛出异常后增强

##### 引介通知

在目标类中添加一些新的方法和属性



#### 引介（Introduction）

引介是一种特殊的通知在不修改类代码的前提下， Introduction可以在运行期为类动态地添加一些方法或属性。

#### 目标（Target）

要被代理的目标对象。

#### 织入（Weaving）

织入指把增强应用到目标对象来创建新的代理对象的过程。

#### 代理（Proxy）

一个类被AOP织入增强后，产生的结果代理类。

#### 切面（Aspect）

是切入点和通知（引介）的结合。



## Spring中的AOP

### Spring中的通知：

前置通知：org.springframework.aop.MethodBeforeAdvice

后置通知：org.springframework.aop.AfterReturningAdvice

环绕通知：org.aopalliance.intercept.MethodInterceptor

异常抛出通知：org.springframework.aop.ThrowsAdvice

引介通知：org.springframework.aop.IntroductionInterceptor

### Spring的切面类型

Advisor：一个切点和一个通知组合。代表一般切面，Advice本身就是一个切面，对目标类所有方法进行拦截。

PointcutAdvisor : 代表具有切点的切面，可以指定拦截目标类哪些方法(带有切点的切面,针对某个方法进行拦截)。

IntroductionAdvisor : 代表引介切面，针对引介通知而使用切面。

Aspect：多个切点和多个通知组合。

#### 不带有切点的切面（Advisor）

##### 导入Maven依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.16.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.3.16.RELEASE</version>
</dependency>
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
<!-- lombok简化POJO类 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
    <scope>provided</scope>
</dependency>
```

##### 创建业务相关接口和实现类

```java
// 业务接口
package com.spring.aop.service;
public interface UserService {
	void add();
	void delete();
	void update();
	void find();
}
// 业务接口实现类
package com.spring.aop.service;
public class UserServiceImpl implements UserService {
	@Override
	public void add() {
		System.out.println("add");
	}
	@Override
	public void delete() {
		System.out.println("delete");
	}
	@Override
	public void update() {
		System.out.println("update");
	}
	@Override
	public void find() {
		System.out.println("find");
	}
}
```

##### 创建增强类

此增强类是一个切点和一个通知组合。代表一般切面，Advice本身就是一个切面，对目标类所有方法进行拦截。

```java
package com.spring.aop.advice;
import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;
/**
 * 前置增强需要实现MethodBeforeAdvice接口
 */
public class SpringBeforeAdvice implements MethodBeforeAdvice {
	@Override
	public void before(Method method, Object[] args, Object target) throws Throwable {
		System.out.println("进行前置增强...");
	}
}
```

##### 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!-- 定义目标对象 -->
	<bean id="userService" class="com.spring.aop.service.UserServiceImpl"/>
	<!-- 定义增强 -->
	<bean id="springBeforeAdvice" class="com.spring.aop.advice.SpringBeforeAdvice"/>
	<!-- 进行代理 -->
	<bean id="userServiceProxy" 	
          class="org.springframework.aop.framework.ProxyFactoryBean">
		<property name="target" ref="userService"/>
		<property name="interceptorNames" value="springBeforeAdvice"/>
	</bean>
</beans>
```

##### 进行测试

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    UserService userService = (UserService) context.getBean("userServiceProxy");
    userService.add();
    userService.delete();
    userService.update();
    userService.find();
}
```

输出结果：

​	进行前置增强...
​	add
​	进行前置增强...
​	delete
​	进行前置增强...
​	update
​	进行前置增强...
​	find

#### 带有切点的切面（PointcutAdvisor ）

带有切点的切面：针对目标对象的某些方法进行增强。

使用Spring提供的org.springframework.aop.support.RegexpMethodPointcutAdvisor类进行定义切点切面：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!-- 定义目标对象 -->
	<bean id="userService" class="com.spring.aop.service.UserServiceImpl"/>
	<!-- 定义增强 -->
	<bean id="springBeforeAdvice" class="com.spring.aop.advice.SpringBeforeAdvice"/>
	<!-- 带有切点的切面 -->
    <bean id="pointcutAdvisor" 
          class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
        <!-- 对add、find进行增强springAdvice -->
		<property name="patterns" value=".*add.*,.*find.*"/>
		<property name="advice" ref="springBeforeAdvice"/>
	</bean>
	<!-- 进行代理 -->
	<bean id="userServiceProxy" 
          class="org.springframework.aop.framework.ProxyFactoryBean">
		<property name="target" ref="userService"/>
		<property name="interceptorNames" value="pointcutAdvisor"/>
	</bean>
</beans>
```

进行测试

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    UserService userService = (UserService) context.getBean("userServiceProxy");
    userService.add();
    userService.delete();
    userService.update();
    userService.find();
}
```

输出结果：

​	进行前置增强...
​	add
​	delete
​	update
​	进行前置增强...
​	find

#### 自动生成代理类

上述的不带有切点切面和带有切点切面的两个例子，都有同样的问题：对每一个Target对象进行代理时，都需要通过ProxyFactoryBean编写相应的一个代理类：userServiceProxy，如果代码中存在很多个Target对象，那么相应的也需要编写多个代理类对象。

自动生成代理类可通过Spring自动生成Target对象的代理类。

自动创建代理基于BeanPostProcessor，在Bean创建的过程中完成的增强，生成的Bean就是代理。

##### 按名称生成代理类

使用org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator完成按名称生成代理类。

###### 配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 定义目标对象 -->
    <bean id="userService" class="com.spring.aop.service.UserServiceImpl"/>
    <!-- 定义增强 -->
    <bean id="springBeforeAdvice" class="com.spring.aop.advice.SpringBeforeAdvice"/>
   	<!-- 自动代理:按名称的代理 基于后处理Bean,后处理Bean不需要配置ID -->
    <bean 
       class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
        <property name="beanNames" value="*Service"/>
        <property name="interceptorNames" value="springBeforeAdvice"/>
    </bean>
</beans>
```

###### 进行测试

```java
@Test
public void test02() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    UserService userService = (UserService) context.getBean("userService");
    userService.add();
    userService.delete();
    userService.update();
    userService.find();
}
```

输出结果：

​	进行前置增强...
​	add
​	进行前置增强...
​	delete
​	进行前置增强...
​	update
​	进行前置增强...
​	find

##### 根据切面中定义的信息生成代理

使用org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator根据切面中定义的信息生成代理。

###### 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!-- 定义目标对象 -->
	<bean id="userService" class="com.spring.aop.service.UserServiceImpl"/>
	<!-- 定义增强 -->
	<bean id="springBeforeAdvice" class="com.spring.aop.advice.SpringBeforeAdvice"/>
	<bean id="pointcutAdvisor" 
          class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
		<property name="patterns" value=".*add.*,.*find.*"/>
		<property name="advice" ref="springBeforeAdvice"/>
	</bean>
	<!-- 自动生成代理 -->
	<bean 
     class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
</beans>
```

###### 进行测试

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    UserService userService = (UserService) context.getBean("userService");
    userService.add();
    userService.delete();
    userService.update();
    userService.find();
}
```

输出结果：

​	进行前置增强...
​	add
​	delete
​	update
​	进行前置增强...
​	find

注：区分基于ProxyFattoryBean的代理与自动代理区别：

ProxyFactoryBean：先有被代理对象,将被代理对象传入到代理类中生成代理。

自动代理基于后处理Bean。在Bean的生成过程中，就产生了代理对象，把代理对象返回，生成Bean已经是代理对象。

### AspectJ

AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法，它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件。

#### AspectJ特点：

​	AspectJ是一个基于Java语言的AOP框架。

​	Spring2.0以后新增了对AspectJ切点表达式支持。

​	@AspectJ 是AspectJ1.5新增功能，通过JDK5注解技术，允许直接在Bean类中定义切面。

新版本Spring框架，建议使用AspectJ方式来开发AOP。

#### AspectJ表达式

execution()是最常用的切点函数，整个表达式可以分为五个部分，其语法如下所示：

execution (\* com.sample.service.impl..*.*(..))

（1）execution(): 表达式主体。

（2）第一个 \*号：表示返回类型，*号表示所有的类型。

（3）包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，com.sample.service.impl包、子孙包下所有类的方法。

（4）第二个\*号：表示类名，\*号表示所有的类。

（5）\*(..)：最后这个星号表示方法名，\*号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数。

execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)

更详细的AspectJ表达式可以通过搜索引擎了解更多。

#### AspectJ增强

@Before 前置通知，相当于BeforeAdvice。

@AfterReturning 后置通知，相当于AfterReturningAdvice。

@Around 环绕通知，相当于MethodInterceptor。

@AfterThrowing抛出通知，相当于ThrowAdvice。

@After 最终final通知，不管是否异常，该通知都会执行。

@DeclareParents 引介通知，相当于IntroductionInterceptor。

#### 基于注解的AspectJ开发

##### 导入Maven依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.16.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.3.16.RELEASE</version>
</dependency>
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
<!-- lombok简化POJO类 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.16.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.13</version>
</dependency>
```

##### 编写被增强的类

```java
package com.spring.aop.aspectj;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
@Aspect
public class AnnotationAspectJ {
	@Before("execution(* com.spring.aop.service.UserService.add(..))")
	public void before() {
		System.out.println("进行前置增强...");
	}	
}
```

##### 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">
	<!-- 底层就是AnnotationAwareAspectJAutoProxyCreator -->
	<aop:aspectj-autoproxy/>
	<!-- 定义目标对象 -->
	<bean id="userService" class="com.spring.aop.service.UserServiceImpl"/>
	<!-- 定义切面对象 -->
	<bean id="annotationAspectJ" class="com.spring.aop.aspectj.AnnotationAspectJ"/>
</beans>
```

##### 进行测试

```java
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans1.xml");
    UserService userService = (UserService) context.getBean("userService");
    userService.add();
    userService.delete();
    userService.update();
    userService.find();
}
```

输出结果：

​	进行前置增强...
​	add
​	delete
​	update
​	find

#### 基于XML的AspectJ开发

##### 编写业务接口和类

```java
package com.spring.aop.service;
public interface AspectJUserService {
	void add();
	String delete();
	void update() throws Exception;
	String find();
}

package com.spring.aop.service;
public class AspectJUserServiceImpl implements AspectJUserService {
	@Override
	public void add() {
		System.out.println("add");
	}
	@Override
	public String delete() {
		System.out.println("delete");
		return "delete";
	}
	@Override
	public void update() throws Exception {
		System.out.println("update");
		throw new Exception("throw Exception...");
	}
	@Override
	public String find() {
		System.out.println("find");
		return "find";
	}
}
```



##### 编写切面

```java
package com.spring.aop.aspectj;

import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class XMLAspectJ {
	public void before() {
		System.out.println("进行前置增强...");
	}
	public void afterReturning(Object returnVal) {
		System.out.println("进行后置增强...");
		System.out.println("后置增强返回值：" + returnVal);
	}
	public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		System.out.println("环绕前增强....");
		Object result = proceedingJoinPoint.proceed();
		System.out.println("环绕后增强....");
		return result;
	}
	public void afterThrowing(Throwable e) {
		System.out.println("异常通知..." + e.getMessage());
	}
}
```

##### 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">
	<!-- 底层就是AnnotationAwareAspectJAutoProxyCreator -->
	<aop:aspectj-autoproxy/>
	<!-- 定义目标对象 -->
	<bean id="aspectJUserService" class="com.spring.aop.service.AspectJUserServiceImpl"/>
	<!-- 定义切面对象 -->
	<bean id="xmlAspectJ" class="com.spring.aop.aspectj.XMLAspectJ"/>
	<aop:config>
		<!-- 定义切点: -->
		<aop:pointcut expression="execution(* 
             com.spring.aop.service.AspectJUserService.add(..))" 
             id="beforePointcut"/>
		<aop:pointcut expression="execution(* 
             com.spring.aop.service.AspectJUserService.delete(..))" 		
             id="afterReturningPointcut"/>
		<aop:pointcut expression="execution(* 
             com.spring.aop.service.AspectJUserService.update(..))" 
             id="afterThrowingPointcut"/>
		<aop:pointcut expression="execution(* 
             com.spring.aop.service.AspectJUserService.find(..))" 
             id="aroundPointcut"/>
		<aop:aspect ref="xmlAspectJ">
			<!-- 前置通知 -->
			<aop:before method="before" pointcut-ref="beforePointcut"/>
			<aop:after-returning method="afterReturning" 
              	pointcut-ref="afterReturningPointcut" returning="returnVal"/>
			<aop:after-throwing method="afterThrowing" 
                pointcut-ref="afterThrowingPointcut" throwing="e"/>
			<aop:around method="around" pointcut-ref="aroundPointcut"/>
		</aop:aspect>
	</aop:config>
</beans>
```

##### 进行测试

```java
@Test
public void test01() throws Exception {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans1.xml");
    AspectJUserService userService = (AspectJUserService) 	
        context.getBean("aspectJUserService");
    userService.add();
    userService.delete();
    userService.find();
    userService.update();
}
```

输出结果：

​	进行前置增强...
​	add
​	delete
​	进行后置增强...
​	后置增强返回值：delete
​	环绕前增强....
​	find
​	环绕增强返回值：find
​	环绕后增强....
​	update
​	异常通知...throw Exception...