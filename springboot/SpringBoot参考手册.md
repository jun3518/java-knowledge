# 第2部分：入门指南

## 11 开发您的第一个SpringBoot应用程序

### 11.1 创建POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.19.RELEASE</version>
    </parent>
    <!-- Additional lines to be added here... -->
</project>
```

### 11.2 添加类路径依赖

Spring Boot提供了许多启动程序（“Starters”），可以方便地将jar添加到类路径。我们的示例应用程序已经在POM的父部分中使用了spring-boot-starter-parent。spring-boot-starter-parent是一个特殊的启动程序，它提供了有用的Maven缺省值。它还提供了一个依赖关系管理部分，这样您就可以为受祝福的依赖关系省略版本标记。

让我们编辑pom.xml，并在父部分下面添加spring-boot-starter-web依赖项：

```xml
<dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

如果运行mvn dependency:tree，您将看到现在有许多附加的依赖项，包括Tomcat web服务器和Spring Boot本身。

### 11.3 编写代码

要完成我们的应用程序，我们需要创建一个Java文件。默认情况下，Maven将从src/main/ java编译源代码，因此需要创建该文件夹结构，然后添加一个名为src/main/ java/Example.java的文件：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {
    
    @RequestMapping("/")
    String home() {
    return "Hello World!";
    }
    
    public static void main(String[] args) throws Exception {
    SpringApplication.run(Example.class, args);
    }
}
```

#### @EnableAutoConfiguration注解

第二个类级注释是@EnableAutoConfiguration。这个注释告诉Spring Boot到“猜测”您想要如何配置Spring，基于您已经添加的jar依赖项。由于Spring -boot-starter-web添加了Tomcat和Spring MVC，因此自动配置将假定您正在开发一个web应用程序并相应地设置Spring。

#### main方法

应用程序的最后一部分是main方法。这只是应用程序入口点遵循Java约定的标准方法。我们的main方法通过调用run将委托给Spring Boot的SpringApplication类。SpringApplication将引导我们的应用程序，启动Spring，而Spring又将启动自动配置的Tomcat web服务器。我们需要将Example.class作为参数传递给run方法，以告诉Spring application哪个是主Spring组件。还传递args数组来公开任何命令行参数。

### 11.5 创建一个可执行jar

要创建一个可执行jar，我们需要将spring-boot-maven-plugin添加到pom.xml中。在dependencies部分下面插入以下行：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

如果查看目标目录，应该会看到myproject-0.0.1-snap.jar。文件大小应该在10 MB左右。如果你想看里面，你可以用jar tvf：

```shell
jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

您还应该看到一个名为myproject-0.0.1-snap.jar的小得多的文件。原始文件在目标目录中。这是Maven在Spring Boot重新打包之前创建的原始jar文件。



# 第3部分：使用SpringBoot

本节将详细介绍如何使用Spring Boot。它涵盖了构建系统、自动配置和如何运行应用程序等主题。我们还介绍了一些Spring Boot的最佳实践。尽管Spring Boot并没有什么特别之处(它只是您可以使用的另一个库)，但是有一些建议，当您遵循这些建议时，将使您的开发过程变得更加简单。

## 13 构建系统

### 13.1 依赖管理

Spring Boot的每个版本都提供了它支持的依赖项列表。实际上，您不需要为构建配置中的任何这些依赖项提供版本，因为Spring Boot正在为您管理这些依赖项。当您升级Spring Boot本身时，这些依赖项也将以一致的方式升级。

注：如果您认为有必要，您仍然可以指定一个版本并覆盖Spring Boot的建议。

### 13.2 Maven

Maven用户可以从spring-boot-starter-parent项目继承，以获得合理的默认值。

父项目提供以下功能：

（1）Java 1.6作为默认编译器级别。

（2）UTF-8编码。

（3）依赖项管理部分，允许您省略公共依赖项的\<version>标记，这些依赖项继承自spring-boot-dependencies POM。

（4）合理的资源过滤。

（5）合理的插件配置。

（6）应用程序的合理资源筛选。application.properties和application.yml。包含特定于概要文件的文件（例如application-foo.properties和application-foo.yml）。

#### 继承启动父级

要将项目配置为继承自spring-boot-starter-parent，只需设置父类：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.19.RELEASE</version>
</parent>
```

注：您应该只需要在此依赖项上指定Spring Boot版本号。如果导入其他starters，可以安全地省略版本号。

使用该设置，您还可以通过覆盖您自己项目中的属性来覆盖各个依赖项。

#### 改变Java版本

spring-boot-starter-parent选择了相当保守的Java兼容性。如果您想遵循我们的建议并使用较高的Java版本，您可以添加一个Java。版本属性：

```xml
<properties>
	<java.version>1.8</java.version>
</properties>
```

#### 使用Spring Boot Maven插件

Spring Boot包含一个Maven插件，可以将项目打包为可执行jar。如果您想使用插件，请将插件添加到您的\<plugins>节点中：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

如果您使用Spring Boot starter父pom，您只需要添加插件，不需要配置它，除非您想更改父pom中定义的设置。

### 13.5 启动器（Starters）

启动器是一组方便的依赖关系描述符，您可以将它们包含在应用程序中。您可以获得所需的所有Spring和相关技术的一站式服务，而不必遍历样例代码并复制粘贴依赖关系描述符的负载。例如，如果您想开始使用Spring和JPA进行数据库访问，只需在项目中包含Spring -boot-starter-data- JPA依赖项，就可以了。

启动程序包含许多依赖项，您需要这些依赖项来快速启动和运行项目，并使用一组一致的、受支持的托管传递依赖项。

注：所有正式启动程序都遵循类似的命名模式;spring-boot-starter-\*，其中\*是一种特定类型的应用程序。这种命名结构是为了在您需要找到初学者时提供帮助。许多IDE中的Maven集成允许您根据名称搜索依赖项。

## 14 构建你的代码

Spring Boot不需要任何特定的代码布局来工作，但是，有一些最佳实践可以提供帮助。

### 14.1 使用“默认”包

当一个类不包含包声明时，它被认为是在“默认包”中。一般不鼓励使用“缺省包”，应该避免使用。对于使用@ComponentScan、@EntityScan或@SpringBootApplication注释，因为每个jar中的每个类都将被读取。

### 14.2 定位主应用程序类

我们通常建议您将主应用程序类定位在根包中，高于其他类。@SpringBootApplication注释通常放在主类上，它隐式地为某些项定义了一个基本的“搜索包”。例如，如果您正在编写JPA应用程序，@SpringBootApplication注释类的包将用于搜索@Entity项。使用根包还允许组件扫描仅应用于您的项目。

注：如果您不想使用@SpringBootApplication，@EnableAutoConfiguration和@ComponentScan它导入的注释定义了该行为，所以您也可以使用它。

## 15 配置类

Spring Boot支持基于Java的配置。虽然可以调用SpringApplication.run()加载XML配置文件，我们通常建议您使用@Configuration类加载Bean。通常，定义main方法的类也可以作为主@Configuration。

注：网上已经发布了许多使用XML配置的Spring配置示例。如果可能，始终尝试使用等效的基于Java的配置。寻找启用Enable\*可能是一个很好的起点。

### 15.1 导入其他配置类

您不需要将所有@Configuration放入一个类中。@Import注解可用于导入其他配置类。或者，您可以使用@ComponentScan自动获取所有Spring组件，包括@Configuration类。

### 15.2 导入XML配置

如果您绝对必须使用基于XML的配置，我们建议您仍然从一个@configuration类开始。然后可以使用附加的@ImportResource注释来加载XML配置文件。

```java
@SpringBootApplication
@ImportResource("config/beans.xml")
public class SpringbootReferrence01Application {
	public static void main(String[] args) {
		SpringApplication.run(SpringbootReferrence01Application.class, args);
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="person" class="com.zhxj.Person">
		<property name="username" value="zhxj"/>
		<property name="password" value="123456"/>
		<property name="hobbits">
			<list>
				<value>吃饭</value>
				<value>睡觉</value>
			</list>
		</property>
	</bean>
</beans>
```



## 16 自动配置

Spring Boot 自动配置尝试根据添加的jar依赖项自动配置Spring应用程序。例如，如果HSQLDB位于您的类路径上，而您还没有手动配置任何数据库连接bean，那么我们将自动配置内存中的数据库。

您需要通过添加@EnableAutoConfiguration或来选择自动配置@Configuration类的@SpringBootApplication注解。您应该只添加一个@SpringBootApplication或@EnableAutoConfiguration注释。我们通常建议您将其中之一添加到您的主要@Configuration类。

### 16.1 逐渐取代自动配置

自动配置是非侵入性的，在任何时候都可以开始定义自己的配置来替换自动配置的特定部分。例如，如果您添加自己的DataSource bean，默认的嵌入式数据库支持将失效。

如果您需要了解当前应用的是什么自动配置，以及为什么，请使用—debug开关启动您的应用程序。这将为选择的核心日志记录器启用调试日志，并将自动配置报告记录到控制台。

### 16.2 禁用特定的自动配置

如果您发现正在应用您不想要的特定自动配置类，您可以使用@EnableAutoConfiguration的exclude属性禁用它们。

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;
@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果类不在类路径上，则可以使用注解的excludeName属性并指定完全限定名。最后，还可以通过spring.autoconfigure.exclude属性控制要排除的自动配置类列表。

注：您可以在注解级别和使用属性定义排除。

## 17 Spring Beans和依赖项注入

您可以自由地使用任何标准Spring框架技术来定义bean及其注入的依赖项。为了简单起见，我们经常使用@ComponentScan来查找bean，与@Autowired构造函数注入结合使用效果很好。

如果按照上面的建议构造代码(将应用程序类定位在根包中)，可以添加@ComponentScan，而不需要任何参数。所有应用程序组件(@Component，@Service、@Repository、@Controller等)将自动注册为Spring bean。

下面是一个示例@Service Bean，它使用构造函数注入来获得所需的RiskAssessor Bean：

```java
package com.example.service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
@Service
public class DatabaseAccountService implements AccountService {
    private final RiskAssessor riskAssessor;
    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
    this.riskAssessor = riskAssessor;
    }
	// ...
}
```

如果bean有一个构造函数，则可以省略@Autowired：

```java
@Service
public class DatabaseAccountService implements AccountService {
	private final RiskAssessor riskAssessor;
    public DatabaseAccountService(RiskAssessor riskAssessor) {
    this.riskAssessor = riskAssessor;
    }
    // ...
}
```

## 18 使用@SpringBootApplication注解

许多Spring Boot开发人员喜欢他们的应用程序使用自动配置、组件扫描，并且能够在他们的“应用程序类”上定义额外的配置。可以使用一个@SpringBootApplication注释来启用这三个特性，即：

（1）@EnableAutoConfiguration：启用Spring Boot的自动配置机制。

（2）@ComponentScan：在应用程序所在的包上启用@Component scan。

（3）@Configuration：允许在上下文中注册额外的bean或导入额外的配置类。

注：@SpringBootApplication注解等价于使用@Configuration、@EnableAutoConfiguration和@ComponentScan及其默认属性。

```java
package com.example.myproject;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {
    public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
    }
}
```

注：@SpringBootApplication还提供别名来定制@EnableAutoConfiguration和@ComponentScan的属性。

注：这些特性都不是强制性的，您可以选择用它支持的任何特性替换这个注释。例如，你可能不想在你的应用程序中使用组件扫描：

```java
package com.example.myproject;
import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
@Configuration
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {
    public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
    }
}
```

在本例中，应用程序与其他任何Spring引导应用程序一样，只是有所不同，不会自动检测到@Component注解的类，并且显式导入用户定义的bean。

## 19 运行你的程序

将应用程序打包为jar并使用嵌入式HTTP服务器的最大优点之一是，可以像运行其他应用程序一样运行应用程序。调试Spring引导应用程序也很容易，您不需要任何特殊的IDE插件或扩展。

注：本节只讨论基于jar的打包，如果选择将应用程序打包为war文件，应该参考服务器和IDE文档。

### 19.2 作为打包的应用程序运行

如果使用Spring Boot Maven或Gradle插件来创建可执行jar，则可以使用java -jar运行应用程序。例如：

```shell
java -jar target/myproject-0.0.1-SNAPSHOT.jar
```

### 19.3 使用Maven插件

Spring Boot Maven插件包含一个运行目标，可用于快速编译和运行应用程序。

```shell
mvn spring-boot:run
```

## 20 开发工具

Spring Boot包含一组额外的工具，可以使应用程序开发体验更愉快。spring-boot-devtools模块可以包含在任何项目中，以提供额外的开发时特性。要包含devtools支持，只需将模块依赖项添加到构建中：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

当运行完全打包的应用程序时，开发人员工具将自动禁用。如果您的应用程序是使用java -jar启动的，或者是使用特殊的类加载器启动的，那么它就被认为是一个生产应用程序。在Maven中将依赖项标记为可选的，这是一种最佳实践，可以防止devtools过渡地应用于使用项目的其他模块。

注：默认情况下，重新打包的归档文件不包含devtools。如果希望使用某些远程devtools特性，需要禁用excludeDevtools build属性来包含它。Maven支持该属性。

### 20.1 属性默认值

Spring Boot支持的几个库使用缓存来提高性能。例如，模板引擎将缓存已编译的模板，以避免重复解析模板文件。此外，Spring MVC可以在提供静态资源时向响应添加HTTP缓存头。

虽然缓存在生产中非常有用，但在开发过程中它可能会产生反效果，使您无法看到刚才在应用程序中所做的更改。因此，spring-boot- devtools默认情况下将禁用这些缓存选项。

缓存选项通常由应用程序中的设置配置。属性文件。例如，Thymeleaf提供spring.thymeleaf。缓存属性。spring-boot-devtools模块将自动应用合理的开发时配置，而不需要手动设置这些属性。

### 20.2 自动重启

使用spring-boot-devtools的应用程序将在类路径上的文件发生更改时自动重启。当在IDE中工作时，这可能是一个有用的特性，因为它为代码更改提供了一个非常快的反馈循环。默认情况下，指向文件夹的类路径上的任何条目都将被监视，以进行更改。注意，某些资源(如静态资产和视图模板)不需要重启应用程序。

注：由于DevTools监视类路径资源，触发重启的惟一方法是更新类路径。更新类路径的方式取决于使用的IDE。在Eclipse中，保存修改后的文件将导致更新类路径并触发重启。在IntelliJ IDEA中，构建项目(Build ->构建项目)将具有相同的效果。

注：DevTools依赖于应用程序上下文的shutdown钩子在重启期间关闭它。如果您禁用了关机钩子(SpringApplication.setRegisterShutdownHook(false))。

注：当决定类路径上的一个条目更改时是否应该触发重启时，DevTools会自动忽略名为spring-boot、spring-boot- DevTools、spring- boot-autoconfigure、spring-boot-actuator和spring-boot-starter的项目。

#### 20.2.1 排除资源

某些资源在被更改时不一定需要重新启动。例如,Thymeleaf模板可以就地编辑。默认情况下，在/META-INF/ maven、/META-INF/resources、/resources、/static、/public或/templates中更改资源不会触发重启，但会触发实时重新加载。如果希望自定义这些排除，可以使用spring.devtools.restart.exclude属性。例如，要只排除/static和/ public，您可以设置如下：

```yaml
spring.devtools.restart.exclude=static/**,public/**
```

#### 20.2.2 监控额外路径

当您更改不在类路径上的文件时，您可能希望重新启动或重新加载应用程序。为此，使用spring.devtools.restart. extrapaths属性配置附加路径，以监视更改。您可以使用上面描述的spring.devtools.restart.exclude属性来控制附加路径下面的更改是会触发完全重启，还是只是实时重新加载。

#### 20.2.3 禁止重启

如果不希望使用restart特性，可以使用spring.devtools.restart.enabled属性禁用它。在大多数情况下，您可以在应用程序中设置这个参数。属性(这仍然初始化restart类加载器，但它不会监视文件更改)。

如果您需要完全禁用restart支持，因为它不能与特定的库一起工作，那么您需要在调用SpringApplication.run(…)之前设置一个系统属性。例如：

```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```



# 第4部分：Spring Boot特性

## 23 SpringApplication

SpringApplication类提供了一种方便的方法来引导将从main()方法启动的Spring应用程序。在许多情况下，您可以只委托给静态对象SpringApplication.run方法:

```java
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```

### 23.1 启动失败

如果您的应用程序启动失败，注册的故障分析器将有机会提供专用的错误消息和修复问题的具体操作。

Spring Boot提供了许多FailureAnalyzer实现，您可以很容易地添加自己的实现。

如果没有故障分析程序能够处理异常，您仍然可以显示完整的自动配置报告，以便更好地理解哪里出错了。为此，您需要启用debug属性或启用debug日志记录org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitialize。

例如，如果使用java -jar运行应用程序，可以启用debug属性，如下所示：

```shell
java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 23.2 自定义Banner

可以通过在类路径中添加一个banner .txt文件或设置banner.location属性来指定Banner文件位置来更改启动时打印的Banner。如果文件有一个不寻常的编码，您可以设置banner.charset指定字符集(默认为UTF-8)。除了文本文件，您还可以添加一个banner.gif、banner.jpg或banner.png图像文件到类路径，或设置banner.image.location属性。图像将被转换成ASCII艺术表示形式，并打印在任何文本横幅上方。

Banner相关变量：

| Variable                         | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| ${application.version}           | 应用程序在MANIFEST.MF中声明的版本号。例如实现版本:1.0打印为1.0。 |
| ${application.formatted-version} | 应用程序在清单中声明的版本号。用于显示的MF格式(用方括号括起来，并以v为前缀)。例如(v1.0)。 |

.................................

如果希望以编程方式生成横幅，可以使用SpringApplication.setBanner(…)方法。使用org.springframework.boot。并实现您自己的printBanner()方法。

如果您想禁用应用程序中的横幅，YAML映射到false，请确保添加引号：

```yaml
spring:
	main:
		banner-mode: "off"
```

### 23.3 定制SpringApplication

如果您不喜欢SpringApplication的默认值，那么您可以创建一个本地实例并定制它。例如，要关掉Banner，你可以这样写：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

注：传递给SpringApplication的构造函数参数是spring bean的配置源。在大多数情况下，这些将是对@Configuration类的引用，但也可以是对XML配置或应该扫描的包的引用。

还可以使用应用程序配置spring应用程序。属性文件。

### 23.5 应用程序事件和监听器

除了通常的Spring框架事件，如ContextRefreshedEvent, SpringApplication也会发送一些附加的应用程序事件。

有些事件实际上是在创建ApplicationContext之前触发的，因此您不能将侦听器注册为@Bean。您可以通过SpringApplication.addListeners(…)或SpringApplicationBuilder.listeners(…)方法序注册它们。

如果希望不管应用程序是如何创建的，都自动注册这些侦听器，那么可以添加META-INF/spring.factories文件到您的项目，并使用org.springframework.context.ApplicationListener的key引用您的侦听器。

```properties
org.springframework.context.ApplicationListener=com.example.project.MyListener
```

当应用程序运行时，应用程序事件按以下顺序发送：

（1）ApplicationStartingEvent在运行开始时发送，但在任何处理之前发送，除了监听器和初始化器的注册。

（2）当在上下文中使用的环境已知时，但在创建上下文中之前，将发送ApplicationEnvironmentPreparedEvent。

（3）ApplicationPreparedEvent将在启动刷新之前，但在加载bean定义之后发送。

（4）在刷新之后将发送一个ApplicationReadyEvent，并处理任何相关的回调，以指示应用程序已准备好为请求提供服务。

（5）如果启动时出现异常，则发送ApplicationFailedEvent。

应用程序事件使用Spring Framework的事件发布机制发送。此机制的一部分确保在子Context中发布给监听器的事件也在任何父上下文中发布给侦听器。因此，如果您的应用程序使用SpringApplication实例的层次结构，监听器可能会接收同一类型应用程序事件的多个实例。

### 23.6 Web环境

一个SpringApplication将尝试代表您创建正确类型的ApplicationContext。默认情况下，将使用AnnotationConfigApplicationContext或AnnotationConfigEmbeddedWebApplicationContext，这取决于您是否正在开发web应用程序。

用于确定“web环境”的算法相当简单(基于几个类的存在)。如果需要覆盖默认值，可以使用setWebEnvironment(boolean webEnvironment)。

还可以通过调用setApplicationContextClass(…)来完全控制将要使用的ApplicationContext类型。

### 23.7 访问应用程序参数

如果需要访问传递给SpringApplication.run()的应用程序参数，可以注入org.springframework.boot.ApplicationArguments Bean实例。ApplicationArguments接口提供对原始String[]参数以及已解析的选项和非选项参数的访问。

```java
import org.springframework.boot.*
import org.springframework.beans.factory.annotation.*
import org.springframework.stereotype.*
@Component
public class MyBean {
    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }
}
```

Spring Boot还将向Spring环境注册一个CommandLinePropertySource。这还允许您使用@Value注解注入单个应用程序参数。

### 23.8 使用ApplicationRunner或CommandLineRunner

如果您需要在SpringApplication启动后运行一些特定的代码，您可以实现ApplicationRunner或CommandLineRunner接口。这两个接口都以相同的方式工作，并提供一个单一的run方法，该方法将在SpringApplication.run(…)完成之前调用。

CommandLineRunner接口以简单的字符串数组的形式提供对应用程序参数的访问，而ApplicationRunner使用上面讨论的ApplicationArguments接口。

```java
import org.springframework.boot.*
import org.springframework.stereotype.*
@Component
public class MyBean implements CommandLineRunner {
    public void run(String... args) {
    	// Do something...
    }
}
```

您还可以实现org.springframework.core.Ordered接口或使用org.springframework.core.annotation..Order注解，如果有多个CommandLineRunner或ApplicationRunner bean的定义必须按照特定的顺序调用。

### 23.9 应用程序退出

每个spring应用程序都将向JVM注册一个退出钩子，以确保ApplicationContext在退出时被优雅地关闭。可以使用所有标准的Spring生命周期回调(例如DisposableBean接口或@PreDestroy注解)。

此外，bean可以实现org.springframework.boot.ExitCodeGenerator接口，如果希望在调用SpringApplication.exit()时返回特定的退出代码，则使用ExitCodeGenerator接口。然后可以将此退出代码传递到System.exit()，将其作为状态代码返回。

```java
@SpringBootApplication
public class ExitCodeApplication {
    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
    	return new ExitCodeGenerator() {
            @Override
            public int getExitCode() {
                return 42;
                }
            };
        }
    
    public static void main(String[] args) {
        System.exit(SpringApplication
        .exit(SpringApplication.run(ExitCodeApplication.class, args)));
    }
}
```

此外，ExitCodeGenerator接口可以由异常实现。当遇到这样的异常时，Spring Boot将返回由实现的getExitCode()方法提供的退出代码。

### 23.10 管理功能

可以通过指定spring.application.admin.enabled属性为应用程序启用与管理相关的特性。这将在MBeanServer平台上公开SpringApplicationAdminMXBean。您可以使用此功能远程管理Spring引导应用程序。这对于任何服务包装器实现都是有用的。

如果您想知道应用程序在哪个HTTP端口上运行，请获取带有key local.server.port的属性。

在启用此功能时要小心，因为MBean公开了一个方法来关闭应用程序。

## 24 外部配置

Spring Boot允许您将配置外部化，这样您就可以在不同的环境中使用相同的应用程序代码。您可以使用属性文件、YAML文件、环境变量和命令行参数来具体化配置。属性值可以使用@Value注释直接注入bean，可以通过Spring环境抽象访问，也可以通过@ConfigurationProperties绑定到结构化对象。

Spring Boot使用一种非常特殊的PropertySource顺序，其设计目的是允许合理地覆盖值。属性按以下顺序考虑：

### 24.1 配置随机值

RandomValuePropertySource用于注入随机值(例如，注入秘密或测试用例)。它可以生成整数、长、uuid或字符串等等。

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

### 24.2 访问命令行属性

默认情况下，SpringApplication将把任何命令行选项参数(以——例如——server.port=9000开始)转换为属性，并将其添加到Spring环境中。如上所述，命令行属性始终优先于其他属性源。

如果不希望将命令行属性添加到环境中，可以使用SpringApplication.setAddCommandLineProperties禁用它们(false)。

### 24.3 应用程序属性文件

SpringApplication将从应用程序加载属性。属性文件位于以下位置，并将其添加到Spring环境中：

（1）当前目录的一个子目录。

（2）当前目录。

（3）类路径/配置包。

（4）根路径。

列表按优先级排序(在列表中较高位置定义的属性覆盖在较低位置定义的属性)。

您还可以使用YAML ('.yml')文件作为'.properties'的替代。

如果你不喜欢application.properties作为配置文件名，您可以通过指定spring.config.name环境属性切换到另一个配置文件名。还可以使用spring.config.location引用显式位置的位置环境属性(以逗号分隔的目录位置或文件路径列表)。

```shell
java -jar myproject.jar --spring.config.name=myproject
```

or

```shell
java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/
override.properties
```

警告：

spring.config.name和spring.config.location在很早以前就被用来确定必须加载哪些文件，因此必须将它们定义为环境属性(通常是OS env、系统属性或命令行参数)。

如果spring.config.location包含目录(与文件相反)，它们应该以/结尾(并将在加载之前附加由spring.config.name生成的名称，包括特定于概要文件的文件名)。在spring.config.location中指定的文件。位置按原样使用，不支持特定于概要文件的变体，并且将被任何特定于概要文件的属性覆盖。

配置位置以相反的顺序搜索。默认情况下，配置的位置是类路径:/、类路径:/config/、文件:./、文件:./config/。搜索结果的顺序为：

（1）file:./config/

（2）file:./

（3）classpath:/config/

（4） classpath:/

配置自定义配置位置时，除了使用默认位置外，还将使用它们。在默认位置之前搜索自定义位置。例如，如果自定义位置classpath:/ custom-config/，file:./custom-config/已配置，搜索顺序变为:

（1）file:./custom-config/

（2）classpath:custom-config/

（3）file:./config/

（4）file:./

（5）classpath:/config/

（6） classpath:/

这种搜索顺序允许您在一个配置文件中指定默认值，然后在另一个配置文件中选择性地覆盖这些值。您可以在应用程序中为您的应用程序提供默认值。属性(或您在spring.config.name中选择的任何其他基本名称)位于一个默认位置。然后，可以在运行时使用位于自定义位置之一的不同文件覆盖这些默认值。

注：如果使用环境变量而不是系统属性，大多数操作系统不允许使用周期分隔的键名，但是可以使用下划线(例如SPRING_CONFIG_NAME而不是spring.config.name)。

### 24.4 Profile-specific属性

/**除了应用属性文件，特定于概要文件的属性也可以使用命名约定application-{profile}.properties来定义。环境有一组默认概要文件(默认情况下[默认])，如果没有设置活动概要文件(即如果没有显式激活概要文件，则使用application-default.properties)。

特定于Profile-specific属性从与标准应用程序相同的位置application.properties，使用特定于概要文件的文件总是覆盖非特定文件，而不管特定于概要文件是在打包的jar内部还是外部。

如果指定了多个概要文件，则应用“最后胜出”策略。例如，spring.profiles.active指定的概要文件。活动属性是在通过SpringApplication配置这些属性之后添加的因此优先考虑API。

**/

### 24.5 属性中的占位符

应用程序中的application.properties在使用时通过现有Environment进行筛选，以便您可以引用以前定义的值(例如，来自系统属性)。

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

### 24.6 使用YAML配置文件替代Properties配置文件

YAML是JSON的一个超集，因此是指定分层配置数据的一种非常方便的格式。当您的类路径上有SnakeYAML库时，SpringApplication类将自动支持YAML作为属性的替代。

#### 24.6.1 加载YAML

Spring框架提供了两个方便的类，可用于加载YAML文档。YamlPropertiesFactoryBean将加载YAML作为属性，而YamlMapFactoryBean将加载YAML作为 Map。

例如：

```yaml
environments:
	dev:
		url: http://dev.bar.com
		name: Developer Setup
	prod:
		url: http://foo.bar.com
		name: My Cool App
```

将转化为以下Properties：

```properties
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App
```

YAML列表用[index] dereferencers表示为属性键，例如这个YAML：

```yaml
my:
	servers:
		- dev.bar.com
		- foo.bar.com
```

#### 24.6.2 在Spring环境中将YAML作为属性公开

YamlPropertySourceLoader类可用于在Spring环境中将YAML公开为属性源。这允许您使用熟悉的带有占位符语法的@Value注释来访问YAML属性。

#### 24.6.3 多配置文件YAML文档

您可以使用spring在一个文件中指定多个特定于概要文件的YAML文档。配置文件键，指示何时应用文档。例如：

```yaml
server:
	address: 192.168.1.100
---
spring:
    profiles: development
server:
	address: 127.0.0.1
---
spring:
	profiles: production
server:
	address: 192.168.1.120
```

在上面的例子中，如果development配置文件处于活动状态，server.address属性将为127.0.0.1。如果没有启用development和productio配置文件，那么属性的值将是192.168.1.100。

如果在应用程序上下文启动时没有显式激活配置文件，则默认配置文件将被激活。在这个YAML中，我们为security.user.password设置了一个值。只有在“默认”配置文件中可用:

```yaml
server:
	port: 8000
---
spring:
	profiles: default
security:
	user:
	password: weak
```

#### 24.6.4 YAML的缺点

无法通过@PropertySource注解加载YAML文件。因此，在需要以这种方式加载值的情况下，需要使用属性文件。

### 24.7 类型安全的配置属性

使用@Value(“${property}”)注解注入配置属性有时会很麻烦，特别是当您处理多个属性或者您的数据本质上是分层的时候。Spring Boot提供了另一种处理属性的方法，允许强类型bean管理和验证应用程序的配置。

```java
package com.example;
import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import org.springframework.boot.context.properties.ConfigurationProperties;
@ConfigurationProperties("foo")
public class FooProperties {
    private boolean enabled;
    private InetAddress remoteAddress;
    private final Security security = new Security();
    public boolean isEnabled() { ... }
    public void setEnabled(boolean enabled) { ... }
    public InetAddress getRemoteAddress() { ... }
    public void setRemoteAddress(InetAddress remoteAddress) { ... }
    public Security getSecurity() { ... }
    public static class Security {
        private String username;
        private String password;
        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));
        public String getUsername() { ... }
        public void setUsername(String username) { ... }
        public String getPassword() { ... }
        public void setPassword(String password) { ... }
        public List<String> getRoles() { ... }
        public void setRoles(List<String> roles) { ... }
	}
}
```

上面的POJO定义了以下属性：

（1） foo.enabled，默认是false。

（2）foo.remote-address，使用可以从字符串强制的类型。

（3）foo.security.username，使用嵌套的“安全性”，其名称由属性的名称确定。特别是返回类型根本没有使用，而且可能已经使用了SecurityProperties。

（4）foo.security.password。

（5） foo.security.roles，字符串集合。

您还需要列出要在@EnableConfigurationProperties注解中注册的属性类：

```java
@Configuration
@EnableConfigurationProperties(FooProperties.class)
public class MyConfiguration {
}
```

当以这种方式注册@ConfigurationProperties bean时，该bean将具有一个常规名称:\<prefix>-\<fqn>，其中\<prefix>是在@ConfigurationProperties注释中指定的环境键前缀，\<fqn>是该bean的完全限定名。如果注释不提供任何前缀，则只使用bean的完全限定名。

即使上面的配置将为FooProperties创建一个常规bean，我们建议@ConfigurationProperties只处理环境，特别是不从上下文注入其他bean。尽管如此，@EnableConfigurationProperties注释也会自动应用到您的项目中，以便使用@ConfigurationProperties注释的任何现有bean都可以从环境中配置。您可以通过确保FooProperties已经是一个bean来简化上面的MyConfiguration。

```java
@Component
@ConfigurationProperties(prefix="foo")
public class FooProperties {
	// ... see above
}
```

这种配置方式在SpringApplication外部YAML配置中特别好用：

```yaml
# application.yml
foo:
	remote-address: 192.168.1.1
	security:
		username: foo
		roles:
            - USER
            - ADMIN
# additional configuration as required
```

要使用@ConfigurationProperties bean，您可以像注入任何其他bean一样注入它们。

```java
@Service
public class MyService {
	private final FooProperties properties;
    @Autowired
    public MyService(FooProperties properties) {
    	this.properties = properties;
    }
    //...
    @PostConstruct
    public void openConnection() {
    	Server server = new Server(this.properties.getRemoteAddress());
    }
} 
```

#### 24.7.1 第三方配置

除了使用@ConfigurationProperties注解类外，还可以在公共@Bean方法上使用该类。当您希望将属性绑定到您无法控制的第三方组件时，这尤其有用。

要从环境属性配置bean，请将@ConfigurationProperties添加到其bean注册：

```java
@ConfigurationProperties(prefix = "bar")
@Bean
public BarComponent barComponent() {
	...
}
```

使用bar前缀定义的任何属性都将以类似于上面的FooProperties示例的方式映射到该BarComponent bean。

#### 27.7.2 松散的绑定

Spring Boot使用一些轻松的规则将环境属性绑定到@ConfigurationProperties bean，因此环境属性名和bean属性名之间不需要精确匹配。有用的常见例子包括虚线分隔(例如上下文路径绑定到contextPath)和大写(例如端口绑定到端口)环境属性。

例如，给定以下@ConfigurationProperties类：

```java
@ConfigurationProperties(prefix="person")
public class OwnerProperties {
    private String firstName;
    public String getFirstName() {
    	return this.firstName;
    }
    public void setFirstName(String firstName) {
    	this.firstName = firstName;
    }
}
```

以下属性名称均可使用：

| 属性                | 说明                                                |
| ------------------- | --------------------------------------------------- |
| person.firstName    | 标准驼峰语法                                        |
| person.first-name   | 虚线表示法，建议在.properties和.yml文件中使用。     |
| person.first_name U | 下划线符号，用于.properties和.yml文件的另一种格式。 |
| PERSON_FIRST_NAME   | 大写格式。建议在使用系统环境变量时使用。            |

#### 27.7.3 属性转换

当Spring绑定到@ConfigurationProperties bean时，它将尝试强制外部应用程序属性为正确的类型。如果需要自定义类型转换，可以提供一个ConversionService bean(使用bean id ConversionService)或自定义属性编辑器(通过CustomEditorConfigurer bean)或自定义转换器(使用注释为@ConfigurationPropertiesBinding的bean定义)。

#### 27.7.4 @ConfigurationProperties验证

每当使用Spring的@Validated注解对类进行标注时，Spring Boot将尝试验证@ConfigurationProperties类。您可以使用JSR-303javax.validation直接在配置类上使用验证约束注解。只需确保兼容的JSR-303实现位于类路径上，然后向字段添加约束注解。

```java
@ConfigurationProperties(prefix="foo")
@Validated
public class FooProperties {
    @NotNull
    private InetAddress remoteAddress;
    // ... getters and setters
}
```

为了验证嵌套属性的值，必须将关联字段标注为@Valid以触发其验证。例如，基于上面的FooProperties例子：

```java
@ConfigurationProperties(prefix="connection")
@Validated
public class FooProperties {
    @NotNull
    private InetAddress remoteAddress;
    @Valid
    private final Security security = new Security();
    // ... getters and setters
    public static class Security {
        @NotEmpty
        public String username;
        // ... getters and setters
    }
}
```

#### @ConfigurationProperties VS @Value

@Value是一个核心容器特性，它不提供与类型安全配置属性相同的特性。下表总结了@ConfigurationProperties和@Value支持的特性：

| 特点       | @ConfigurationProperties | @Value |
| ---------- | ------------------------ | ------ |
| 松散的绑定 | Yes                      | No     |
|            | Yes                      | No     |
| SpEL       | No                       | Yes    |

如果您为自己的组件定义了一组配置键，我们建议您将它们分组到一个带有@ConfigurationProperties注解的POJO中。还请注意，由于@Value不支持松散绑定，如果您需要使用环境变量提供值，那么它不是一个很好的选择。

注：虽然可以在@Value中编写SpEL表达式，但是这些表达式并不从应用程序属性文件中处理。

## 25 Profiles

Spring Profiles提供了一种方法来隔离应用程序配置的各个部分，并使其仅在某些环境中可用。任何@Component或@Configuration都可以用@Profile标记，以限制加载它的时间。

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {
	// ...
}
```

在正常的Spring方法中，您可以使用spring.profiles.active环境属性，以指定哪些profiles是有效的。您可以使用任何常用方法指定该属性，例如可以将其包含在application.properties中

```yaml
spring.profiles.active=dev,hsqldb
```

或者在命令行上使用--spring.profile .active=dev,hsqldb指定。

### 25.1 添加可用的Profiles

spring.profiles.active属性遵循与其他属性相同的顺序规则，最高的PropertySource将获胜。这意味着您可以在application.properties中指定活动profiles。然后使用命令行开关替换它们。

有时，将特定于profile的属性添加到活动profile中而不是替换它们是有用的。spring.profiles.include 属性可用于无条件地添加活动profile。SpringApplication入口点还有一个用于设置其他概要文件的Java API(例如，在那些被spring.profiles.active属性):参见setAdditionalProfiles()方法。

例如，当使用--spring.profiles运行具有以下属性的应用程序时。proddb和prodmq概要文件也将被激活：

```yaml
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
    - proddb
    - prodmq
```

记住那个spring.profiles属性可以在YAML文档中定义profiles属性，以确定何时将此特定文档包含在配置中。

### 25.2 以编程方式设置profiles

您可以通过调用SpringApplication以编程方式设置活动profiles。在应用程序运行之前执行SpringApplication.setAdditionalProfiles(…)。还可以使用Spring的ConfigurableEnvironment接口激活配置文件。

## 26 日志

Spring Boot对所有内部日志使用Commons Logging，但保留底层日志实现。为 Java Util Logging，Log4J2 和Logback提供了默认配置。在每种情况下，日志记录器都预先配置为使用控制台输出，同时还提供可选的文件输出。

默认情况下，如果使用启动程序，将使用Logback进行日志记录。还包括适当的Logback路由，以确保使用Java Util Logging、Commons Logging、Log4J或SLF4J的依赖库都能正确工作。

Java有很多可用的日志框架。如果上面的列表看起来令人困惑，不要担心。一般情况下，您不需要更改日志依赖项，并且Spring引导默认值将正常工作。

### 26.1 日志格式

Spring Boot的默认日志输出如下所示：

```properties
2014-03-05 10:57:51.112 INFO 45469 --- [ main] org.apache.catalina.core.StandardEngine :
Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253 INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/] :
Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253 INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader :
Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698 INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean :
Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702 INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean :
Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

下面是输出项：

（1）日期和时间 - 毫秒精度和易于排序。

（2）日志级别 - ERROR、WARN、INFO、DEBUG、TRACE。

（3）Process ID（进程id）。

（4）--- 分隔符，以区分实际日志消息的开始。

（5）用方括号括起来的线程名(可能为控制台输出而被截断)。

（6）日志的名字 - 这通常是源类名(通常缩写)。

（7）日志消息。

注：Logback没有致命级别(它被映射为ERROR)。

### 26.2 控制台输出

默认的日志配置将在消息写入时将其打印到控制台。默认情况下，ERROR、WARN和INFO级别的消息将被记录。您还可以通过使用--debug标志启动应用程序来启用调试模式。

```shell
$ java -jar myapp.jar --debug
```

还可以在application.properties中指定debug=true。

启用调试模式后，将配置一系列核心日志记录器(嵌入式容器、Hibernate和Spring Boot)，以输出更多信息。启用调试模式并不会将应用程序配置为记录所有具有调试级别的消息。

#### 26.2.1 彩色编码输出

如果您的终端支持ANSI，颜色输出将用于帮助可读性。你可以设置spring.output.ansienabled.enabled启用到受支持的值以覆盖自动检测。

颜色编码是使用%clr转换字配置的。例如，在最简单的形式中，转换器将根据日志级别对输出进行着色：

```properties
%clr(%5p)
```

日志级别到颜色的映射如下所示：

| 日志级别 | 颜色   |
| -------- | ------ |
| FATAL    | Red    |
| ERROR    | Red    |
| WARN     | Yellow |
| INFO     | Green  |
| DEBUG    | Green  |
| TRACE    | Green  |

或者，您可以指定应该使用的颜色或样式，将其作为转换的选项提供。例如，将文本设置为黄色：

```properties
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式：blue、cyan、faint、green、red、yellow。

### 23.6 文件输出

默认情况下，Spring Boot只会登录到控制台，不会写入日志文件。如果您想在控制台输出之外编写日志文件，则需要设置logging.file或logging.path属性(例如在application.properties中)。

下表显示了如何进行日志记录。属性可以一起使用：

| logging.file | logging.path | 例子     | 描述                                                         |
| ------------ | ------------ | -------- | ------------------------------------------------------------ |
| (none)       | (none)       |          | 在控制台输出                                                 |
| 指定文件     | (none)       | my.log   | 写入指定的日志文件。名称可以是确切的位置或相对于当前目录。   |
| (none)       | 指定目录     | /var/log | 将spring.log写入指定的目录。名称可以是确切的位置或相对于当前目录。 |

日志文件在达到10mb时将会旋转，与控制台输出一样，默认情况下会记录错误、警告和信息级别的消息。

日志系统在应用程序生命周期的早期初始化，因此在通过@PropertySource注释加载的属性文件中不会找到此类日志属性。

日志属性独立于实际的日志基础设施。因此，特定的配置键(如Logback的logback.configurationFile)。不是由spring Boot管理的。

### 26.4 日志级别

所有受支持的日志系统都可以使用log .level在Spring环境中设置日志记录器级别(例如application.properties)。使用logging.level.*=LEVEL，其中LEVEL是TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF中的一个。可以使用log.level.root配置根日志程序。例子application.properties：

```properties
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

### 26.5 自定义日志配置

可以通过在类路径中包含适当的库来激活各种日志系统，并通过在类路径的根目录中或Spring Environment属性logging.config指定的位置中提供适当的配置文件来进一步定制。

您可以使用org.springframework.boot.logging.LoggingSystem系统属性强制Spring Boot使用特定的日志系统。该值应该是LoggingSystem实现的完全限定类名。您还可以使用none值完全禁用Spring Boot的日志配置。

由于日志是在创建ApplicationContext之前初始化的，所以不可能控制来自Spring @Configuration文件中的@PropertySources的日志记录。更改日志系统或完全禁用它的唯一方法是通过系统属性。

根据您的日志系统，将加载以下文件：

| 日志系统                | 自定义                                                       |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | logback-spring.xml, logback-spring.groovy, logback.xml or logback.groovy |
| Log4j2                  | log4j2-spring.xml or log4j2.xml                              |
| JDK (Java Util Logging) | logging.properties                                           |

如果可能，我们建议您在日志配置中使用-spring变体(例如logback-spring.xml而不是logback.xml)。如果使用标准配置位置，Spring无法完全控制日志初始化。

Java Util Logging中存在一些已知的类加载问题，这些问题会在从可执行jar运行时引发问题。我们建议你尽可能避免使用它。

为了帮助进行定制，将其他一些属性从Spring环境转移到系统属性：

| Spring环境                        | 系统属性                      | 说明                                                         |
| --------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| logging.exception-conversion-word | LOG_EXCEPTION_CONVERSION_WORD | 记录异常时使用的转换字                                       |
| logging.file                      | LOG_FILE                      | 如果定义，则在默认日志配置中使用                             |
| logging.path                      | LOG_PATH                      | 如果定义，则在默认日志配置中使用                             |
| logging.pattern.console           | CONSOLE_LOG_PATTERN           | 要在控制台(stdout)上使用的日志模式。(只支持默认的logback设置。) |
| logging.pattern.file              | FILE_LOG_PATTERN              | 要在文件中使用的日志模式(如果启用了LOG_FILE)。(只支持默认的logback设置。) |
| logging.pattern.level             | LOG_LEVEL_PATTERN             | 用于呈现日志级别的格式(默认%5p)。(只支持默认的logback设置。) |
| PID                               | PID                           | 当前进程ID(如果可能，在尚未定义为OS环境变量时发现)           |

支持的所有日志系统在解析配置文件时都可以参考系统属性。有关示例，请参阅spring-boot.jar中的默认配置。

如果希望在日志属性中使用占位符，应该使用Spring Boot的语法，而不是底层框架的语法。值得注意的是，如果您正在使用Logback，您应该使用：作为属性名称与其默认值之间的分隔符，而不是:-. 。

您可以通过只覆盖LOG_LEVEL_PATTERN(或logging.pattern)，将MDC和其他特别的内容添加到日志行。和Logback水平)。例如，如果您使用log.pattern.level=user:%X{user} %5p如果存在，那么默认日志格式将包含“user”的MDC条目，例如：

```verilog
2015-09-30 12:30:04.031 user:juergen INFO 22174 --- [ nio-8080-exec-0] demo.Controller
Handling authenticated request
```

### 26.6 Logback扩展

Spring Boot包含许多Logback扩展，可以帮助进行高级配置。您可以在您的logback-spring.xml配置文件中使用这些扩展。

您不能在标准的logback .xml配置文件中使用扩展名，因为它加载得太早了。您需要使用logback-spring.xml或定义日志logging.config属性。

#### 26.6.1 特殊Profile配置

\<springProfile>标记允许您根据活动的Spring配置文件选择性地包含或排除配置部分。profile部分在\<configuration>元素的任何位置都受支持。使用name属性指定哪个配置文件接受配置。可以使用逗号分隔的列表指定多个profile：

```xml
<springProfile name="staging">
<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>
<springProfile name="dev, staging">
<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>
<springProfile name="!production">
<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

#### 26.6.2 环境属性

\<springProperty>标记允许您在Logback中显示Spring环境中的属性。如果您希望从应用程序访问值，这将非常有用。返回配置中的属性文件。标记的工作方式与Logback的标准\<property>标记类似，但是您指定的是属性的源(来自环境)，而不是直接指定值。如果需要将属性存储在本地范围之外的其他地方，可以使用scope属性。如果在环境中没有设置属性时需要回退值，可以使用defaultValue属性。

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
	defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```

RelaxedPropertyResolver用于访问环境属性。如果使用虚线表示法指定源(my-property-name)，那么将尝试所有轻松的变体(myPropertyName、MY_PROPERTY_NAME等)。

## 27 开发Web应用程序

Spring Boot非常适合web应用程序开发。可以使用嵌入式Tomcat、Jetty或Undertow轻松创建自包含HTTP服务器。大多数web应用程序将使用spring-boot- starter-web模块来快速启动和运行。

### 27.1 Spring Web MVC框架

Spring Web MVC框架(通常简称为Spring MVC)是一个富模型视图控制器Web框架。Spring MVC允许您创建特殊的@Controller或@RestController bean来处理传入的HTTP请求。控制器中的方法使用@RequestMapping注释映射到HTTP。

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {
    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
    	// ...
    }
    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
   	 	// ...
    }
@RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
    	// ...
    }
}
```

#### 27.1.1 Spring MVC自动配置

Spring Boot为Spring MVC提供了自动配置，可以很好地与大多数应用程序配合使用。

自动配置在Spring的默认值之上添加了以下特性：

（1）包含ContentNegotiatingViewResolver和BeanNameViewResolver bean。

（2）支持提供静态资源，包括对webjar的支持。

（3）Converter、GenericConverter、Formatter bean的自动注册。

（4）支持HttpMessageConverters。

（5）MessageCodesResolver的自动注册。

（6）静态index.html支持。

（7）自定义Favicon支持。

（8）自动使用ConfigurableWebBindingInitializer bean。

如果您想保留Spring Boot MVC特性，并且只想添加额外的MVC配置(interceptors, formatters, view controllers等)，您可以添加自己的@Configuration类，类型为WebMvcConfigurerAdapter，但是不需要@EnableWebMvc。如果希望提供RequestMappingHandlerMapping、RequestMappingHandlerAdapter或ExceptionHandlerExceptionResolver的自定义实例，可以声明一个提供此类组件的WebMvcRegistrationsAdapter实例。

如果您想完全控制Spring MVC，您可以添加自己的@Configuration，并使用@EnableWebMvc进行注释。

#### 27.1.2 HttpMessageConverters

Spring MVC使用HttpMessageConverter接口来转换HTTP请求和响应。合理的默认值是开箱即用的，例如对象可以自动转换为JSON(使用Jackson库)或XML(如果可用，使用Jackson XML扩展，否则使用JAXB)。默认情况下，字符串使用UTF-8编码。

如果需要添加或自定义转换器，可以使用Spring Boot的HttpMessageConverters类

```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;
@Configuration
public class MyConfiguration {
    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }
}
```

上下文中出现的任何HttpMessageConverter bean都将添加到转换器列表中。您还可以用这种方式覆盖默认转换器。

#### 27.1.3 自定义JSON序列化器和反序列化器

如果您正在使用Jackson对JSON数据进行序列化和反序列化，您可能希望编写自己的JsonSerializer和JsonDeserializer类。定制序列化器通常通过模块在Jackson中注册，但是Spring Boot提供了另一个@JsonComponent注释，这使得直接注册Spring bean更加容易。

您可以直接在JsonSerializer或JsonDeserializer实现上使用@JsonComponent。您还可以在包含序列化器/反序列化器的类上作为内部类使用它。例如：

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;
@JsonComponent
public class Example {
    public static class Serializer extends JsonSerializer<SomeObject> {
    	// ...
    }
    public static class Deserializer extends JsonDeserializer<SomeObject> {
    	// ...
    }
}
```

ApplicationContext中的所有@JsonComponent bean都将自动向Jackson注册，由于@JsonComponent是用@Component进行元注释的，所以通常使用的组件扫描规则是适用的。

Spring Boot还提供了JsonObjectSerializer和JsonObjectDeserializer基类，它们在序列化对象时提供了标准Jackson版本的有用替代方法。

#### 27.1.4 MessageCodesResolver

Spring MVC有一个策略，用于生成错误代码，以便从绑定错误中呈现错误消息:如果您设置spring.mvc.message-codes-resolver.format属性PREFIX_ERROR_CODE or POSTFIX_ERROR_CODE（参见DefaultMessageCodesResolver.Format中的枚举），Spring Boot会为您创建一个MessageCodesResolver。

### 27.3 嵌入式servlet容器支持

Spring Boot包含对嵌入式Tomcat、Jetty和Undertow服务器的支持。大多数开发人员将简单地使用适当的启动程序来获得完全配置的实例。默认情况下，嵌入式服务器将侦听端口8080上的HTTP请求。

#### 27.3.1 Servlets, Filters, and Listeners

当使用嵌入式servlet容器时，您可以通过使用Spring bean或扫描servlet组件注册servlet规范中的servlet、filter和所有listener(例如HttpSessionListener)。

##### 将servlet、filter和listener注册为Spring bean

任何Servlet、Filter或Servlet *Listener实例(即Spring bean)都将在嵌入式容器中注册。如果希望引用应用程序中的值，这将特别方便。在配置属性。

默认情况下，如果上下文只包含一个Servlet，它将被映射到/。在多个Servlet bean的情况下，bean名称将用作路径前缀。过滤器将映射到/*。

如果基于约定的映射不够灵活，可以使用ServletRegistrationBean、FilterRegistrationBean和ServletListenerRegistrationBean类来完成控制。

#### 27.3.2 Servlet Context初始化

嵌入式servlet容器不会直接执行servlet 3.0+的javax.servlet.ServletContainerInitializer接口，或Spring的org.springframework.web.WebApplicationInitializer接口。这是一个有意的设计决策，旨在降低设计在war中运行的第三方库破坏Spring引导应用程序的风险。

如果需要在Spring引导应用程序中执行servlet上下文初始化，应该注册一个实现org.springframework.boot.context.embedded.ServletContextInitializer的bean单一的onStartup方法提供了对ServletContext的访问，如果需要，可以很容易地用作现有WebApplicationInitializer的适配器。

##### 扫描Servlets, Filters, and listeners

当使用嵌入式容器时，可以使用@ServletComponentScan启用@WebServlet、@WebFilter和@WebListener注释类的自动注册。

@ServletComponentScan在独立容器中不起作用，而是使用容器的内置发现机制。

#### 27.3.4 自定义嵌入式Servlet容器

可以使用Spring环境属性配置常见的servlet容器设置。通常在应用程序中定义application.properties文件。

常见的服务器设置包括：

（1）网络设置：监听传入HTTP请求的端口(server.port)、要绑定到服务器的接口server.address,等等。

（2）会话设置：session是否持久(server.session.persistence)、session超时(server.session.store-dir)，session数据的位置以及session-cookie配置(server.session.cookie.*)。

（3）错误管理：错误页面的位置(server.error.path)，等等。

（4）SSL

（5）HTTP压缩

##### 编程定制

如果需要以编程方式配置嵌入式servlet容器，可以注册一个实现EmbeddedServletContainerCustomizer接口的Spring bean。EmbeddedServletContainerCustomizer提供对ConfigurableEmbeddedServletContainer的访问，该容器包含许多定制setter方法。

```java
import org.springframework.boot.context.embedded.*;
import org.springframework.stereotype.Component;
@Component
public class CustomizationBean implements EmbeddedServletContainerCustomizer {
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
    	container.setPort(9000);
    }
}
```

##### 直接自定义ConfigurableEmbeddedServletContainer

如果上面的定制技术太有限，您可以自己注册TomcatEmbeddedServletContainerFactory、JettyEmbeddedServletContainerFactory或UndertowEmbeddedServletContainerFactory bean。

```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
	TomcatEmbeddedServletContainerFactory factory = new 			
        TomcatEmbeddedServletContainerFactory();
	factory.setPort(9000);
	factory.setSessionTimeout(10, TimeUnit.MINUTES);
	factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
	return factory;
}
```

#### 27.3.5 JSP局限性

当运行使用嵌入式servlet容器(并打包为可执行归档文件)的Spring引导应用程序时，JSP支持存在一些限制。

（1）对于Jetty和Tomcat，如果使用war打包，它应该可以工作。当使用java -jar启动可执行war时，它将工作，并且可以部署到任何标准容器中。使用可执行jar时不支持jsp。

（2）Undertow不支持jsp。

（3）创建自定义error.jsp页面不会覆盖用于错误处理的默认视图，应该使用自定义错误页面。

## 28 安全

