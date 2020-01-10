# SpringBoot笔记
　如果要想在 Spring 之中整合 RabbitMQ、Kafka、ActiveMQ、MySQL、 Druid、Redis、Shiro，需要编写一堆堆的*.xml 配置文件； 所以在这样的一个大的历史背景下，很多人开始寻求更加简便的开发，而遗憾的是这种简便的开发没有被 JDK 所支持、没有 被 JavaEE 所支持，因为这些只是平台，平台能够提供的只是最原始的技术支持。这一时刻终于由于 Spring 框架的升级而得到了新 生，SpringBoot 的出现，改变了所有 Java 开发的困境，SpringBoot 的最终奉行的宗旨：废除掉所有复杂的开发，废除掉所有的配置文件，让开发变得更简单纯粹，核心：“零配置”
## SpringBoot特性
* 快速创建项目，简化配置
* 容器内嵌，独立运行，内嵌了tomcat
* 提供约定的starter POM来简化Maven配置
* 基本可以不使用xml配置文件，只需要注解配置
* 自动配置（spring/springmvc/事务）
## SpringBoot配置文件
### 结构
yml文件，树状结构  
注意点：  
1，原有的key，例如spring.jpa.properties.hibernate.dialect，按“.”分割，都变成树状的配置  
2，key后面的冒号，后面一定要跟一个空格  
3，把原有的application.properties删掉。然后一定要执行一下  maven -X clean install  
### 优先级
application.yml先加载，application.properties后加载，所有后者的优先级高于前者的优先级
## 启动过程
![启动过程](image/20190328185435263.png)
## 注解
### SpringBootApplication
典型的SpringBoot的启动类配置在src/main/java的根路径下。其中@SpringbootApplication开启组件扫描以及自动配置，而SpringApplication.run()则启动引导应用程序
#### @SpringBootConfiguration
可以看作**@Configuration**注解，告诉容器这个是个JavaConfig的配置类，它是Spring框架的注解
#### @ComponentScan
组件扫描，类似于**<context:component-scan>**,如果扫描到有@Controller，@Service，@Component注解的类，则将其配置为Bean.  
Spring全自动扫描所有通过注解配置的bean，然后将其注册到IOC容器中，我们可以通过basepackages等属性来指定扫描范围，如果不指定的话，默认从声明@ComponentScan所在类的包中进行扫描，正因为如此，SpringBoot的启动类都默认在src/main/java下
#### @EnableAutoConfiguration
表示开启SpringBoot自动配置功能，SpringBoot会根据应用的依赖，自定义Bean，classpath下有没有这个类来猜测你需要的bean
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```  
其中最关键的要属[@Import(AutoConfigurationImportSelector.class)]，借助[AutoConfigurationImportSelector],@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建的IOC容器中去  
借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，@EnableAutoConfiguration可以智能的自动配置功效才得以生效，在AutoConfigurationImportSelector类中可以看到通过[SpringFactoriesLoader.loadFactoryNames()]把 [spring-boot-autoconfigure.jar/META-INF/spring.factories]中每一个xxxAutoConfiguration文件都加载到容器中  
SpringFactoriesLoader属于Spring框架私有的一种扩展方案,，其主要功能就是从指定的配置文件META-INF/spring-factories加载配置，spring-factories是一个典型的java properties文件，只不过Key和Value都是Java类型的完整类名,在@EnableAutoConfiguration场景中，它更多提供了一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfig.EnableAutoConfiguration作为查找的Key，获得对应的一组@Configuration类。SpringFactoriesLoader是一个抽象类，类中定义的静态属性定义了其加载资源的路径public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"  
### Import
注解用于导入配置类中，假如在一个Configuration中，需要使用另一个Configuration，这个配置类中有一个Bean依赖于另一个Configuration，则需要使用[@Import]引用  
需要注意的是，在4.2之前[@Import]注解只支持导入配置类，但是在4.2之后，它支持导入普通类，将这个类作为Bean定义注册到IOC容器中。
### Conditional
注解表示满足某个条件之后才会初始化一个Bean或者启用某些配置之后，它一般用在@Component，@Service，@Controller，@Configuration等注解标识的类上面或者由@Bean标记的方法上面。如果一个@Configuration类标记了[@Conditional],则其下的所有@Bean方法以及@Import都将遵从这个条件
* @ConditionalOnClass ： classpath中存在该类时起效
* @ConditionalOnMissingClass ： classpath中不存在该类时起效
* @ConditionalOnBean ： DI容器中存在该类型Bean时起效
* @ConditionalOnMissingBean ： DI容器中不存在该类型Bean时起效
* @ConditionalOnSingleCandidate ： DI容器中该类型Bean只有一个或@Primary的只有一个时起效
* @ConditionalOnExpression ： SpEL表达式结果为true时
* @ConditionalOnProperty ： 参数设置或者值一致时起效
* @ConditionalOnResource ： 指定的文件存在时起效
* @ConditionalOnJndi ： 指定的JNDI存在时起效
* @ConditionalOnJava ： 指定的Java版本存在时起效
* @ConditionalOnWebApplication ： Web应用环境下起效
* @ConditionalOnNotWebApplication ： 非Web应用环境下起效
在Spring里面，可以很方便的编写自己的条件类，所要做的就是实现Condition接口并覆盖matches()方法即可
                                                                                                                                                        


