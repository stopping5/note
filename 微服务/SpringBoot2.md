# 一、什么是Starter

Starter 是 Spring Boot 中的一个非常重要的概念，Starter相当于模块，它能将模块所需的依赖整合起来并对模块内的 Bean 根据环境（ 条件）进行自动配置。**使用者只需要依赖相应功能的 Starter，无需做过多的配置和依赖，Spring Boot 就能自动扫描并加载相应的模块**。实现**开箱即用**。在上一节课中，我们在 Maven 的依赖中加入 spring-bootstarter-web 就能使项目支持 Spring MVC，并且 Spring Boot 还为我们做了很多默认配置，无需再依赖 spring-web、spring-webmvc 等相关包及做相关配置就能够立即使用起来SpringBoot 存在很多开箱即用的 Starter 依赖，使得我们在开发业务代码时能够非常方便的、不需要过多关注框架的配置，而只需要关注业务即可

# 二、手写Starter

## 1、自定义Starter

编写一个自动格式化的starter，实现在Springboot中直接添加依赖就可以直接使用无需任何配置。

提供String和Json两种方法

```java
//接口
public interface FormatProcessor {
    //定义一个格式化的方法
    <T> String format(T obj);
}
//Json实现
public class JsonFormatProcessor implements FormatProcessor{
    @Override
    public <T> String format(T obj) {
        return "JsonFormatProcessor:"+ JSON.toJSONString(obj);
    }
}
//String实现
public class StringFormatProcessor implements FormatProcessor{
    @Override
    public <T> String format(T obj) {
        return "StringFormatProcessor:"+Objects.toString(obj);
    }
}
```

编写模板方法

```java
public class StoppingFormatTemplate {
    private FormatProcessor formatProcessor;

    public StoppingFormatTemplate(FormatProcessor formatProcessor) {
        this.formatProcessor = formatProcessor;
    }

    public <T>String doFormat(T obj){
        StringBuilder stringBuilder=new StringBuilder();
        stringBuilder.append("Obj format result:").append(formatProcessor.format(obj)).append("<br/>");
        return stringBuilder.toString();
    }
}
```

对格式化方法进行配置。这里FormatProcessor有两个实现，所以对其进行条件注册

```java
@Configuration
public class FormatAutoConfiguration {
    //如果没有com.alibaba.fastjson.JSON加载该对象
    @ConditionalOnMissingClass("com.alibaba.fastjson.JSON")
    @Bean
    @Primary
    public StringFormatProcessor stringFormatProcessor(){
        return new StringFormatProcessor();
    }
	//如果有com.alibaba.fastjson.JSON加载该对象
    @ConditionalOnClass(name = "com.alibaba.fastjson.JSON")
    @Bean
    public JsonFormatProcessor jsonFormatProcessor(){
        return new JsonFormatProcessor();
    }
}
```

StoppingAutoConfiguration实现自动装配。导入FormatAutoConfiguration对象，完成StoppingFormatTemplate的注册。这样客户端就可以拿到StoppingFormatTemplate的对象实习自动注入了。

```java
@Configuration
@Import(FormatAutoConfiguration.class)
public class StoppingAutoConfiguration {
    @Bean
    public StoppingFormatTemplate stoppingFormatTemplate(FormatProcessor formatProcessor){
        return new StoppingFormatTemplate(formatProcessor);
    }
}
```

最后，将自动装配的类添加到spring.properties文件中，实现SPI

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.example.autoconfiguration.StoppingAutoConfiguration
```

![image-20200601032650157](H:\TyporaImage\咕泡学习\微服务\SpringBoot2\image-20200601032650157.png)

将文件夹命名META-INF,新建文件spring.properties

![image-20200601032744572](H:\TyporaImage\咕泡学习\微服务\SpringBoot2\image-20200601032744572.png)

将本starter打包。

![image-20200601032812776](H:\TyporaImage\咕泡学习\微服务\SpringBoot2\image-20200601032812776.png)

将依赖信息作为jar包导入到springboot项目中

```
<groupId>org.example</groupId>
<artifactId>StoppingFormat-spring-boot-starter</artifactId>
<version>1.0-SNAPSHOT</version>
```

Springboot项目中进行调用

```java
@RestController
public class HelloController {
    @Autowired
    private StoppingFormatTemplate stoppingFormatTemplate;

    @GetMapping("hello")
    public String hello(){
        User user = new User();
        user.setAge("18");
        user.setName("xdp");
         //如果没有实现自动装配的话
    	// FormatProcessor fp = new JsonFormatProcessor();
        // StoppingFormatTemplate sf = new StoppingFormatTemplate(fp); 
   		// sf.doFormat(user);
        return stoppingFormatTemplate.doFormat(user);
    }
}
```

测试结果：

如有导入了fastJson包则Obj format result:{"age":"18","name":"xdp"}；

没有导入：Obj format result:User{name='xdp', age='18'}

## 2、提供外部配置

类似springboot在application.properties中可以配置相应的key-value。在starter中可以获取使用。

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <version>2.1.6.RELEASE</version>
    <optional>true</optional> 
</dependency>
```

2、创建属性类

```java
//声明属性前缀
@ConfigurationProperties(prefix = StoppingProperties.PREFIX)
public class StoppingProperties {
    //配置属性的前缀
    static final String PREFIX = "stopping.format";
    //属性id
    public Map<String,String> info;
    public Map<String, String> getInfo() {
        return info;
    }
    public void setInfo(Map<String, String> info) {
        this.info = info;
    }
}
```

2、装配到需要用到的地方

```java
public class StoppingFormatTemplate {
    private FormatProcessor formatProcessor;
	//属性
    private StoppingProperties stoppingProperties;

    public StoppingFormatTemplate(StoppingProperties stoppingProperties,FormatProcessor formatProcessor) {
        this.formatProcessor = formatProcessor;
        this.stoppingProperties = stoppingProperties;
    }

    public <T>String doFormat(T obj){
        StringBuilder stringBuilder=new StringBuilder();
        stringBuilder.append("properties:").append(stoppingProperties.getInfo());
        stringBuilder.append("Obj format result:").append(formatProcessor.format(obj)).append("<br/>");
        return stringBuilder.toString();
    }
}
```

3、启用配置@EnableConfigurationProperties(StoppingProperties.class)

```java
@Configuration
@Import(FormatAutoConfiguration.class)
@EnableConfigurationProperties(StoppingProperties.class) //在这里启用了
public class StoppingAutoConfiguration {

    @Bean 
    public StoppingFormatTemplate stoppingFormatTemplate(StoppingProperties stoppingProperties,FormatProcessor formatProcessor){
        return new StoppingFormatTemplate(stoppingProperties,formatProcessor);
    }
}
```

## 3、查看mybatis-starter



# 三、spring-boot-starter-logging





# 四、配置多数据源



# 五、Actuator



# 六、JMX





