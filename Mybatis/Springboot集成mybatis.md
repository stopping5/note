# 一、项目初始化
## 1、引入依赖
```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
```

## 2、配置mybatis
```yml
server:
  port: 8080
spring:
  #数据库连接配置
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password:

#mybatis的相关配置
mybatis:
  #mapper配置文件
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.springbootwithmybatis.pojo
  #开启驼峰命名
  configuration:
    map-underscore-to-camel-case: true
```

==注意==

如果需要配置mybatis-config.xml配置文件的话，springboot配置文件中只能声明mybatis-config.xml文件的位置。

报错信息：

```
nested exception is java.lang.IllegalStateException: Property 'configuration' and 'configLocation' can not specified with together
```

处理方式：mybatis和springboot配置只能选择一种

application.properties

```properties
#mybatis的相关配置
mybatis:
  config-location: classpath:mybatis.cfg.xml
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-mybatis.orgDTD Config 3.0EN"
        "http:mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <setting name="defaultStatementTimeout" value="300" />
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="cacheEnabled" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.example.springbootwithmybatis.pojo"/>
    </typeAliases>
    <typeHandlers>
        <package name="com.example.springbootwithmybatis.type"/>
    </typeHandlers>
    <!-- 配置mybatis的分页插件PageHelper -->
    <!--<plugins>
        <plugin interceptor="com.example.springbootwithmybatis.plugin.ExcutePlugin"></plugin>
    </plugins>-->

    <mappers>
        <package name="com.example.springbootwithmybatis.mapper"/>
    </mappers>
</configuration>
```



# 二、编写映射文件

## 1、新建实体类
```java
package com.example.springbootwithmybatis.pojo;

import lombok.Data;

/**
 * @Author: stopping
 * @Date: 2020/09/11 10:11
 * 转载注明出处、个人博客网站:www.stopping.top
 */
@Data
public class User {
    private Integer id;
    private String name;
    private Integer age;
    private String email;
}
```

## 2、新建mapper接口
**注意：**springboot需要扫描到mapper文件注册到bean管理器中，所以我们要告诉spring那些是mapper文件。具体有两种方式：
1、在mapper接口使用注解@Mapper声明
```java
package com.example.springbootwithmybatis.mapper;

import com.example.springbootwithmybatis.pojo.User;

import java.util.List;

/**
 * @Author: stopping
 * @Date: 2020/09/11 10:11
 * 转载注明出处、个人博客网站:www.stopping.top
 */
//方式一
@Mapper
public interface UserMapper {
    List<User> findAllUser();
}
```
方式二：在springboot启动入口使用@MapperScan("com.example.springbootwithmybatis.mapper")声明mapper所在的位置，这种方式更方便。
```java
@SpringBootApplication
@MapperScan("com.example.springbootwithmybatis.mapper")
public class SpringbootwithmybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootwithmybatisApplication.class, args);
    }

}
```

## 3、编写映射xml文件
将mapper映射文件保存在resources-mapper文件夹中。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.example.springbootwithmybatis.mapper.UserMapper">
	<!--id 对应mapper接口的方法名称 resultType是返回值的类型-->
    <select id="findAllUser" resultType="com.example.springbootwithmybatis.pojo.User">
        select * from  user
    </select>
</mapper>
```
# 三、测试接口
服务层：
```java
package com.example.springbootwithmybatis.service;

import com.example.springbootwithmybatis.mapper.UserMapper;
import com.example.springbootwithmybatis.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

/**
 * @Author: stopping
 * @Date: 2020/09/11 10:11
 * 转载注明出处、个人博客网站:www.stopping.top
 */
@Service
public class UserServiceImp implements UserService{
    @Resource
    private UserMapper userMapper;

    @Override
    public List<User> findAllUser() {
        return userMapper.findAllUser();
    }
}
```
控制层：
```java
package com.example.springbootwithmybatis.controller;

import com.example.springbootwithmybatis.pojo.User;
import com.example.springbootwithmybatis.service.UserService;
import com.example.springbootwithmybatis.service.UserServiceImp;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * @Author: stopping
 * @Date: 2020/09/11 10:11
 * 转载注明出处、个人博客网站:www.stopping.top
 */
@RestController
public class UserController {
    @Autowired
    private UserServiceImp userService;

    @RequestMapping("/findUser")
    public List<User> findUser(){
        return userService.findAllUser();
    }
}

```

# 四、访问测试
![图片alt](img/5f5c8fd1258c7c0fe117835c ''图片title'')