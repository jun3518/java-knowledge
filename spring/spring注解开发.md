### 1、Configuration

```java
//@Configuration声明一个配置类，告诉Spring这是一个配置类，等价于配置文件
@Configuration
public class BeanConfig {

    @Bean
    public UserBean userBean() {
        return new UserBean();
    }
}
```



### 2、Bean

```java
@Configuration
public class BeanConfig {

    // 在配置类中注册bean，默认bean的id是方法名，如果需要指定名称，可以通过@Bean("name")指定
    @Bean
    public UserBean userBean() {
        return new UserBean();
    }
}
```

### 3、ComponentScan

```java
/** 
@ComponentScan:告诉Spring要扫描的包路径，Spring会自动将包下（及子包）的Bean扫描加入Spring容器中。
重要的属性：
	（1）excludeFilters：按照扫描规则，排除指定的组件。
	（2）includeFilters：按照扫描规则，只扫描指定的组件。
*/
@Configuration
@ComponentScan("com.spring")
public class BeanConfig {

    @Bean
    public UserBean userBean() {
        return new UserBean();
    }
}
/**
打印结果：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
beanConfig
userController
userService
userBean
*/

// excludeFilters：按照扫描规则，排除指定的组件。示例：
@Configuration
@ComponentScan(value="com.spring", excludeFilters = {
    @Filter(type=FilterType.ANNOTATION, classes=Controller.class)
})
public class BeanConfig {

    @Bean
    public UserBean userBean() {
        return new UserBean();
    }
}
/**
打印结果：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
beanConfig
userService
userBean

从打印结果来看，userController没有被加载到Spring容器中
*/

// includeFilters：按照扫描规则，只扫描指定的组件。示例：
@Configuration
@ComponentScan(value="com.spring", includeFilters = {
    @Filter(type=FilterType.ANNOTATION, classes=Controller.class)
})
public class BeanConfig {

    @Bean
    public UserBean userBean() {
        return new UserBean();
    }
}
/**
打印结果：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
beanConfig
userController
userService
userBean

从打印结果来看，userController被加载到Spring容器中，且在com.spring包下的所有bean都被加载了。
*/

// 同时声明includeFilters、excludeFilters。示例：
@Configuration
@ComponentScan(value = "com.spring", 
		includeFilters = {
				@Filter(type = FilterType.ANNOTATION, classes = Controller.class) 
		}, 
		excludeFilters = {
				@Filter(type = FilterType.ANNOTATION, classes = Controller.class) 
		}
)
public class BeanConfig {

    @Bean
    public UserBean userBean() {
        return new UserBean();
    }
}
/**
打印结果：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
beanConfig
userService
userBean

从打印结果来看，userController没有被加载到Spring容器中。即优先级excludeFilters > includeFilters
*/
```

### 4、ComponentScans

```java
// ComponentScans注解可以用于包含多个ComponentScan
@Configuration
@ComponentScans(
    value= {
        @ComponentScan(value = "com.spring", 
                       includeFilters = {
                           @Filter(type = FilterType.ANNOTATION, 
                                   classes = Controller.class) 
                       }, 
                       excludeFilters = {
                           @Filter(type = FilterType.ANNOTATION, 
                                   classes = Controller.class) 
                       }
                      ),
        @ComponentScan(value = "com.spring", 
                       includeFilters = {
                           @Filter(type = FilterType.ANNOTATION, 
                                   classes = Controller.class) 
                       }, 
                       excludeFilters = {
                           @Filter(type = FilterType.ANNOTATION, 
                                   classes = Controller.class) 
                       }
                      )
    }
)
public class BeanConfig {

	@Bean
	public UserBean userBean() {
		return new UserBean("zhangsan", "lisi");
	}

}

```

### 5、Scope

```java
public class BeanConfig {
	/**
	可以指定singleton、prototype，
	已经web使用的：request、session。
	其中singleton的实例会在Spring容器启动时进行实例化放进容器中
	*/
    @Scope("singleton")
    @Bean
    public UserBean userBean() {
        return new UserBean("zhangsan", "lisi");
    }

}
```

### 6、Conditional

Conditional注解，用于类中组件统一设置/方法的判断加载。满足当前条件，这个类中配置的所有bean注册才能生效。

用法：

（1）在方法级别使用：

```java
// 定义如果操作系统是Linux，则加载该bean
public class LinuxCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		
		Environment environment = context.getEnvironment();
		String os = environment.getProperty("os.name");
		if(os.contains("Linux")) {
			return true;
		}
		
		return false;
	}
}
// 定义如果操作系统是Windows，则加载该bean
public class WindowsCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		
		Environment environment = context.getEnvironment();
		String os = environment.getProperty("os.name");
		if(os.contains("Windows")) {
			return true;
		}
		
		return false;
	}

}

@Configuration
public class BeanConfig {

    @Conditional(WindowsCondition.class)
    @Bean("bill")
    public UserBean userBean01() {
        return new UserBean("Bill Gates", "123456");
    }

    @Conditional(LinuxCondition.class)
    @Bean("linus")
    public UserBean userBean() {
        return new UserBean("Linus", "123456");
    }
}
```

（2）在类级别使用：当符合条件是，该配置类中的所有Bean定义才会加载。

```java
@Conditional(WindowsCondition.class)
@Configuration
public class BeanConfig {

	@Conditional(WindowsCondition.class)
	@Bean("bill")
	public UserBean userBean01() {
		return new UserBean("Bill Gates", "123456");
	}
}
```

### Import

Import注解可以快速地在Spring容器中导入一个组件，其中该组件的id为类的全路径。

```java
// 在Spring中导入Color组件
@Import(Color.class)
@Configuration
public class BeanConfig2 {

}
```

### ImportSelector

自定义返回需要导入的组件。返回值就是要导入到容器中的组件全类名。

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {

    //返回值，就是到导入到容器中的组件全类名
    //AnnotationMetadata:当前标注@Import注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 建议使用 类名.class.getName() 方式获取类的全类名
        //importingClassMetadata
        //方法不要返回null值
        return new String[]{"com.atguigu.bean.Blue","com.atguigu.bean.Yellow"};
    }
}

@Import({MyImportSelector.class})
@Configuration
public class BeanConfig2 {

}
```

### ImportBeanDefinitionRegistrar

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    /**
	 * AnnotationMetadata：当前类的注解信息
	 * BeanDefinitionRegistry:BeanDefinition注册类；
	 * 		把所有需要添加到容器中的bean；调用
	 * 		BeanDefinitionRegistry.registerBeanDefinition手工注册进来
	 */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, 
                                        BeanDefinitionRegistry registry) {

        boolean definition = registry.containsBeanDefinition("com.spring.beans.Red");
        boolean definition2 = registry.containsBeanDefinition("com.spring.beans.Blue");
        if(definition && definition2){
            //指定Bean定义信息；（Bean的类型，Bean。。。）
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个Bean，指定bean名
            registry.registerBeanDefinition("rainBow", beanDefinition);
        }
    }
}

@Import({MyImportBeanDefinitionRegistrar.class})
@Configuration
public class BeanConfig2 {

}
```

### Bean的初始化和销毁1：initMethod和destroyMethod

```java
public class Car {

    public Car(){
        System.out.println("car constructor...");
    }

    public void init(){
        System.out.println("car ... init...");
    }

    public void detory(){
        System.out.println("car ... detory...");
    }

}

@Configuration
public class BeanConfig3 {
	// 指定初始化方法为init，销毁方法为detory
	@Bean(initMethod="init",destroyMethod="detory")
	public Car car(){
		return new Car();
	}

}
```

### Bean的初始化和销毁2：@PostConstruct和@PreDestroy

```java
public class Car {
	
	public Car(){
		System.out.println("car constructor...");
	}
	
	@PostConstruct
	public void init(){
		System.out.println("car ... init...");
	}
	
	@PreDestroy
	public void detory(){
		System.out.println("car ... detory...");
	}

}

@Configuration
public class BeanConfig3 {
    
    @Bean
	public Car car(){
		return new Car();
	}

}
```

注：初始化和销毁对于prototype实例来说，初始化会进行，但是销毁不会。原因简单：prototype实例不归Spring管理。需要主动调用销毁操作。

### Bean的初始化和销毁3：InitializingBean和DisposableBean

```java
public class Car implements InitializingBean, DisposableBean {
	
	public Car(){
		System.out.println("car constructor...");
	}

	@Override
	public void destroy() throws Exception {
		System.out.println("car ... detory...");
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("car ... init...");
	}

}

@Configuration
public class BeanConfig3 {
    
    @Bean
	public Car car(){
		return new Car();
	}

}
```

### Bean的初始化和销毁4：XML配置：init-method和destroy-method

```xml
<bean id="car" class="com.atguigu.bean.Car" init-method="init" destroy-method="detory"/>
```

### PropertySource/@PropertySources加载外部文件

```java
@PropertySource("classpath:/db.properties")
@Configuration
public class BeanConfig4 {
    
}
    
```











