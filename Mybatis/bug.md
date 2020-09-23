# SpringBoot整合Mybatis报错:You must configure either the server or JDBC driver (via the serverTimezone conf)



## 一、报错描述

```java	
Fri May 17 14:55:39 CST 2019
There was an unexpected error (type=Internal Server Error, status=500).
nested exception is org.apache.ibatis.exceptions.PersistenceException: ### Error querying database. Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLSyntaxErrorException: Identifier name 'netsafe&useunicode=true&characterencoding=utf-8&allowmultiqueries=true&usessl=false' is too long ### The error may exist in com/infosec/netsafess/mapper/UserMapper.java (best guess) ### The error may involve com.infosec.netsafess.mapper.UserMapper.getAllUser ### The error occurred while executing a query ### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLSyntaxErrorException: Identifier name 'netsafe&useunicode=true&characterencoding=utf-8&allowmultiqueries=true&usessl=false' is too long
org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLSyntaxErrorException: Identifier name 'netsafe&useunicode=true&characterencoding=utf-8&allowmultiqueries=true&usessl=false' is too long
### The error may exist in com/infosec/netsafess/mapper/UserMapper.java (best guess)
### The error may involve com.infosec.netsafess.mapper.UserMapper.getAllUser
### The error occurred while executing a query
### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLSyntaxErrorException: Identifier name 'netsafe&useunicode=true&characterencoding=utf-8&allowmultiqueries=true&usessl=false' is too long
	at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:77)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:446)
	at com.sun.proxy.$Proxy60.selectList(Unknown Source)
	at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:230)
	at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:147)
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:80)
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:58)
	at com.sun.proxy.$Proxy64.getAllUser(Unknown Source)
	at com.infosec.netsafess.service.impl.UserServiceImpl.getAllUser(UserServiceImpl.java:19)
	at com.infosec.netsafess.controller.HelloController.getAllUser(HelloController.java:25)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
```

mybatis配置信息

```yml
spring:
  #数据库连接配置
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?characterEncoding=utf-8&useSSL=false
    username: root
    password:
```

启动springboot项目，然后访问数据库，报You must configure either the server or JDBC driver (via the serverTimezone conf）的错误，==原因==:因为安装mysql的时候时区设置的不正确 mysql默认的是美国的时区，而我们中国大陆要比他们迟8小时，采用+8:00格式。使用的数据库是MySQL，从上面图看出SpringBoot2.1在你没有指定MySQL驱动版本的情况下它自动依赖的驱动是8.0.12很高的版本，这是由于数据库和系统时区差异所造成的，在jdbc连接的url后面加上serverTimezone=GMT即可解决问题，如果需要使用gmt+8时区，需要写成GMT%2B8，否则会被解析为空。

## 二、解决方案

### 1、降低驱动版本

### 2、添加时区信息

```
    url: jdbc:mysql://localhost:3306/test?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
```

