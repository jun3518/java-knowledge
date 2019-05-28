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

### 23 . 5 应用程序事件和监听器

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







