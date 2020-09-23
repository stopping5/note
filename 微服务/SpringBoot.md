

# SpringBoot

微框架(SpringBoot)快速构架一个Web应用。对于 spring 框架，我们接触得比较多的应该是 spring mvc、 和 spring。而 spring 的核心在于 IOC（控制反转）和 DI（依赖注入）。而这些框架在使用的过程中会需要配置大量的 xml，或者需要做很多繁琐的配置。**springboot **框架是为了能够帮助使用 spring 框架的开发者快速高效的构建一个**基于 spirng 框架以及 spring 生态体系的应用解决方案**。它是对**“约定优于配置”**这个理念下的一个最佳实践。因此它是一个服务于框架的框架，服务的范围是**简化配置文件**。

# 一、约定优于配置

1、maven的目录结构，默认有 resources 文件夹存放配置文件

2、默认打包方式为 jar

3、**spring-boot-starter-web **中默认包含 spring mvc 相关依赖以及内置的 tomcat 容器，使得构建一个 web 应用更加简单。

4、默认提供 application.properties/yml 文件

5、默认通过 spring.profiles.active 属性来决定运行环境时读取的配置文件。（自动装配）

6、EnableAutoConfiguration 默认对于依赖的 starter 进行自动装载

# 二、SpringBoot 特点（其他基于商spring）

## 1、AutoConfiguration 自动装配

### @SpringBootApplication

SpringBootApplication是一个复合的注解代码如下

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited //是否可继承
@SpringBootConfiguration //本质是@Configuration
@EnableAutoConfiguration //启动自动装配
@ComponentScan(//静态装配@Controller、@Service、@component
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

@SpringBootApplication 其本质是由三个注解组成：

#### @Configuration/@SpringBootConfiguration

它是 JavaConfig形式的基于 Spring IOC 容器的配置类使用的一种注解。因为 SpringBoot 本质上就是一个 spring 应用，所以通过这个注解来加载 IOC 容器的配置是很正常的。所以在启动类里面标注了@Configuration，意味着它其实也是一个 **IoC容器的配置类**。用于声明bean定义的来源。

与其相配套使用的是**@Bean**

相当于XML配置<bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">。在Spring IOC容器中，自动装配使用到bean名称是默认是类名小写作为key，实例对象作为value存放在Map容器中。在@Bean注解下的实例化对象将会自动扫描进入IOC容器，例如：

```java
public class XdpMain {
    public static void main(String[] args) {
        //注意这里 ConfigurationDemo.class
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(ConfigurationDemo.class);
        String [] beanName = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String name:beanName
             ) {
            System.out.println(name);
        }
    }
}


@Configuration
@Import(SpringConfiguration.class)//导入其他包的配置类
public class ConfigurationDemo {
    @Bean
    public ConfigTest configTest(){
      return  new ConfigTest();
    }
}

//等价于
<beans>
    <bean id="configTest" class="com.gupaoedu.springboot.ConfigTest"/> 
</beans>

//输出结果
configurationDemo
com.gupaoedu.springboot.stopping.other.SpringConfiguration
jdbctest//import导入
configTest
```

#### @ComponentScan

它的主要作用就是扫描指定路径下的标识了需要装配的类，自动装配到 spring 的 Ioc 容器中。扫描的标志主要有**@Component / @Repository / @Service /@Controller** 所标志的类。ComponentScan 默认会扫描当前 package 下的的所有加了相关注解标识的类到 IoC 容器中。相当于配置文件的<context:component-scan basepageback = "">

例如扫描com.gupaoedu.springboot.stopping包路径下所有带**@Component / @Repository / @Service /@Controller** 的类。

![image-20200531134238023](H:\TyporaImage\咕泡学习\微服务\SpringBoot\image-20200531134238023.png)

```java
@ComponentScan(basePackages = "com.gupaoedu.springboot.stopping")//默认是当前类所在的包下面的注解 
public class XdpMain {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(XdpMain.class);
        String [] beanName = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String name:beanName
             ) {
            System.out.println(name);//输出扫描到的bean的名称
        }
    }
}
//serviceDemo
@Service
public class ServiceDemo {
}
//controllerDemo
@Controller
public class ControllerDemo {
}
//repository
@Repository
public class RepositoryDemo {
}
//输出结果
xdpMain
controllerDemo
repositoryDemo
serviceDemo
```



#### @EnableAutoConfiguration

主要作用其实就是帮助springboot 应用把所有符合条件的@Configuration 配置都加载到当前 SpringBoot 创建并使用的 IoC 容器中。EnableAutoConfiguration 是一种动态加载配置的注解。其内部组成注解是：@AutoConfigurationPackage / @Import

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

##### @Import

xml 形式下有一个<import resource/> 形式的注解，就明白它的作用了。import 就是把多个分来的容器配置合并在一个配置中。在JavaConfig 中所表达的意义是一样的。

@Import 注解可以配置三种不同的 class

1. 第一种就是前面演示过的，基于普通 bean 或者带有@Configuration 的 bean 进行诸如

2. 实现 ImportSelector 接口进行动态注入

3. 实现 ImportBeanDefinitionRegistrar 接口进行动态注入

```java
@Import({AutoConfigurationImportSelector.class})
```

##### SpringBoot动态装配的两种实现方式

以上的装配机制过于固定无法实现动态选择我们需要的配置。所以下面将介绍springboot如何实现动态的装配机制。即使SPI。

1、实现ImportSelector接口

```java
//实现ImportSelector接口
public class CacheTestImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //通过注解元数据获取注解实例
        Map<String,Object> attributes=
                annotationMetadata.getAnnotationAttributes(EnableDenifiedService.class.getName());
        //动态注入bean :自己去实现判断逻辑实现动态配置
        return new String[]{CacheTestService.class.getName()}; //返回的是一个固定的CacheService
    }
}

//新建注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({CacheTestImportSelector.class}) //加载CacheTestImportSelector
public  @interface EnableDenifiedService {
    //配置一些方法
    Class<?>[] exclude() default {};
}

//使用EnableDenifiedService测试、模仿Springboot中@EnableAutoConfiguration
@SpringBootApplication
c
public class DynamicMain {
    public static void main(String[] args) {
        ConfigurableApplicationContext ca= SpringApplication.run(DynamicMain.class,args);

        ca.getBean(CacheTestService.class).say();//输出语句hello
    }c
}

//如果@EnableDenifiedService(exclude = {CacheTestService.class})
我们能够在CacheTestImportSelector获取exclude信息进行处理、从而实现动态的加载配置。
@com.gupaoedu.springboot.stopping.dontaozhuru.EnableDenifiedService(exclude=[class com.gupaoedu.springboot.stopping.dontaozhuru.CacheTestService])
```

2、实现ImportBeanDefinitionRegistrar 

```java
//实现ImportBeanDefinitionRegistrar
public class LoggerRegister implements ImportBeanDefinitionRegistrar {
    //注册bean的方法
    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        Class bean =  LoggerService.class;
        //通过RootBeanDefinition包装bean
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(bean);
        //beanName 是类名小写
        String name = StringUtils.uncapitalize(bean.getName());
        //注册bean
        beanDefinitionRegistry.registerBeanDefinition(name,rootBeanDefinition);
    }
}

//LoggerService
public class LoggerService {
    public void say(){
        System.out.println("hello i am LoggerService");
    }
}

//添加注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({LoggerRegister.class}) //导入注册bean的配置
public  @interface EnableDenifiedService {
    //配置一些方法
    Class<?>[] exclude() default {};
}

//启动
@SpringBootApplication
@EnableDenifiedService
public class DynamicMain {
    public static void main(String[] args) {
        ConfigurableApplicationContext ca= SpringApplication.run(DynamicMain.class,args);
        ca.getBean(LoggerService.class).say();//hello i am LoggerService
    }
}


```

### Springboot实现自动装配

1. AutoConfigurationImportSelector实现ImportSelector

```java
AutoConfigurationImportSelector.java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        //加载配置spring-autoconfigure-metadata.properties 加载元数据其中包括条件注册内容
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
        //获得配置的实体，将元数据传入
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
        //将配置类名小写装入
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

```properties
#部分spring-autoconfigure-metadata.properties内容
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration.AutoConfigureAfter=org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration.ConditionalOnClass=javax.sql.DataSource,org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration=
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration.Configuration=
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration=
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration.ConditionalOnBean=com.mongodb.reactivestreams.client.MongoClient
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration.ConditionalOnClass=com.hazelcast.core.HazelcastInstance,org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean
org.springframework.boot.autoconfigure.session.MongoReactiveSessionConfiguration.Configuration=
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration.Configuration=
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration.ConditionalOnSingleCandidate=org.springframework.mail.javamail.JavaMailSenderImpl
org.springframework.boot.autoconfigure.session.RedisSessionConfiguration.ConditionalOnClass=org.springframework.data.redis.core.RedisTemplate,org.springframework.session.data.redis.RedisOperationsSessionRepository
org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration.ConditionalOnClass=reactor.core.publisher.Flux,reactor.core.publisher.Mono
```

getAutoConfigurationEntry 加载实体的代码

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        //
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        configurations = this.removeDuplicates(configurations);
        //获得所有被拆除的类
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        //移除
        configurations.removeAll(exclusions);
        //通过元数据过滤
        configurations = this.filter(configurations, autoConfigurationMetadata);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```

getCandidateConfigurations方法，获取需要装配的类

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    //this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader()本接口的类加载器
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

List<String>是META-INF/spring.factories所有key对应的value。这里知识部分内容，我们可以看到这里的key就是EnableAutoConfiguration。vvlaue需要自动装配的类

```properties
# Auto Configure @EnableAutoConfiguration 这个接口约定的配置信息
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
```

在loadFactoryNames方法中，他会通过List<String>提供的key去找META-INF/spring.factories文件中的value

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}
//classLoader 接口名 即 key 
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
    if (result != null) {
        return result;
    } else {
        try {
            Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
            LinkedMultiValueMap result = new LinkedMultiValueMap();
                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryClassName = ((String)entry.getKey()).trim();
                        String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        int var10 = var9.length;

                        for(int var11 = 0; var11 < var10; ++var11) {
                            String factoryName = var9[var11];
                            result.add(factoryClassName, factoryName.trim());
                        }
                    }
                }

                cache.put(classLoader, result);
        }
    }
```

### 条件注解

在元数据注册的时候有一个文件META-INF/spring-autoconfigure-metadata.properties。在SpringBoot入口处可以注解使用。 

```java
public static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
    return loadMetadata(classLoader, "META-INF/spring-autoconfigure-metadata.properties");
}
```

例如

```
com.gupaoedu.core.GupaoConfig.ConditionalOnClass=com.gupaoedu.TestClass

```

表示在com.gupaoedu.core.GupaoConfig路径下如果存在com.gupaoedu.TestClass才加载。

例如在Springboot中

```properties
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration.ConditionalOnBean=com.mongodb.reactivestreams.client.MongoClient
```

加载mongo信息的时候需要有MongoClient才能加载，一定程度的解释了Springboot的开箱即用的原理。只有你用到的组件的时候才会自动装配信息。



2、@AutoConfigurationPackage import Register 实现ImportBeanDefinitionRegistrar

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```

```java
//Register.java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AutoConfigurationPackages.register(registry, (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
    }

    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new AutoConfigurationPackages.PackageImport(metadata));
    }
}
```







2、Starter

3、Actuator

4、SpringBoot CLI

# 四、SPI

## 1、SPI是什么

SPI ，全称为 Service Provider Interface，是一种服务发现机制。它通过在ClassPath路径下的META-INF/services文件夹查找文件，自动加载文件里所定义的类。这一机制为很多框架扩展提供了可能，比如在Dubbo、JDBC中都使用到了SPI机制。我们先通过一个很简单的例子来看下它是怎么用的。https://www.jianshu.com/p/3a3edbcd8f24

## 2、案例

 h'g

# 五、**深入理解条件过滤** 

在分析 AutoConfigurationImportSelector 的源码时，会先扫描 spring-autoconfiguration-metadata.properties文件，最后在扫描 spring.factories 对应的类时，会结合前面的元数据进行过滤，为什么要过滤呢？ 原因是很多的@Configuration 其实是依托于其他的框架来加载的，如果当前的 classpath 环境下没有相关联的依赖，则意味着这些类没必要进行加载，所以，通过这种条件过滤可以有效的减少@configuration 类的数量从而降低SpringBoot 的启动时间。

![image-20200531214102793](H:\TyporaImage\咕泡学习\微服务\SpringBoot\image-20200531214102793.png)





# 五、学习方法

1、3w1h 来知道学习一个新的东西->初步认识->场景->需求->解决方案->应用->原理

2、官网一定会有quick start 快速入门。通过demo知道他能做什么

3、将知识沉淀成自己的知识。