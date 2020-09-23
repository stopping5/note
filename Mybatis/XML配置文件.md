# 一、Mybatis核心对象生命周期
## 1、核心对象
### SqlSessionFactoryBuilder
这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。
### SqlSessionFactory
SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。
### Session
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。 下面的示例就是一个确保 SqlSession 关闭的标准模式：
### Mapper
映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的。虽然从技术层面上来讲，任何映射器实例的最大作用域与请求它们的 SqlSession 相同。但方法作用域才是映射器实例的最合适的作用域。 也就是说，映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃。 映射器实例并不需要被显式地关闭。尽管在整个请求作用域保留映射器实例不会有什么问题，但是你很快会发现，在这个作用域上管理太多像 SqlSession 的资源会让你忙不过来。 因此，最好将映射器放在方法作用域内。就像下面的例子一样：

# 二、Mybatis执行流程
![图片alt](https://i.loli.net/2020/09/16/shmIpyH5DMbN96S.png ''图片title'')

# 三、mybatis基本配置
## 1、mybatis-config.xml基本配置
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--读取外部配置文件-->
    <properties resource="db.properties"></properties>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/><!-- 单独使用时配置成MANAGED没有事务 -->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.user}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="user.xml"></mapper>
    </mappers>
</configuration>
```

## 2、db.properties数据源信息
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.user=root
jdbc.password=
```

## 3、新建Mapper接口类，即dao层
```
public interface UserDao {
    //声明接口
    List<User> selectUser();
}
```

## 4、user.xml文件配置
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jdbc.dao.UserDao">//此处声明了UserDao的位置
    <resultMap id="BaseUser" type="com.jdbc.pojo.User">//resultMap需要声明对象和数据库表的映射关系
        <id column="id" property="id" jdbcType="BIGINT"></id>
        <result column="name" property="name" jdbcType="VARCHAR"></result>
    </resultMap>

    <select id="selectUser" resultMap="BaseUser">//selectUser 对应userDao接口方法名
        select * from USER
     </select>
</mapper>
```

## 5、测试
```
public class UserDaoTest {
    @Test
    public void testConnection() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        //通过调用sqlSession封装的方法获取
        List<User> users = sqlSession.selectList("com.jdbc.dao.UserDao.selectUser");
        for (User u : users) {
            System.out.println("获取姓名"+u.getName());
        }
	}
}
```

# 四、XML映射文件详解
本文使用实体类有User、Blog。代码如下：
```
public class Blog implements Serializable {
    private int id;
    private String title;
    private String content;
    private String tag;
    private User user;
	//get set
}

public class User implements Serializable {
    private Long id;
    private String name;
    private int age;
    private List<Blog> blogs;
	//get set
}

```
## 1、select查询语句
### (1)代码示例
```
	//user.xml
	<select id="selectUserById" resultType="com.jdbc.pojo.User">
		select * from User u where u.id = #{userId}
	</select>
	//userDao.java
	User selectUserById(Integer userId);
```
### （2）配置解析
表示查询语句通过用户id查询得到的结果映射到User实体类上。#{userId},userId是selectUserById的参数，#{} MyBatis 创建一个预处理语句（PreparedStatement）参数。
```
// 近似的 JDBC 代码，非 MyBatis 代码...
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,userId);
```
注意与${}的区别，${}是对其进行替换。

### (3)常用属性
| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 唯一标识符，被引用时调用                                     |
| resultType    | 返回的对象                                                   |
| resultMap     | resultMap结果集的命名，与resultType只能用其中一个            |
| parameterType | 对于传入本语句的参数的类名                                   |
| userCache     | 使用二级缓存**select默认开启缓存**                           |
| flushCache    | 清空二级缓存，默认false                                      |
| fetchSize     | 返回数据的行数                                               |
| statementType | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |


## 2、insert、update、delete 修改元素
### (1)代码示例
### (2)代码解释
### (3)常用属性

## 3、sql 用于常定义的sql语句
可以将一些比较高频的部分sql进行替代。还可以使用参数注入。

### （1）代码演示

#### 1、基本用法
``` xml
	<sql id="selectSql">
        selecy * from user u
    </sql>
    <select id="selectUserById" resultType="com.jdbc.pojo.User"> 
	<include refid="selectSql"></include>
        from User u where u.id = #{userId}
    </select>
```

#### 2、参数注入
```xml
    <sql id="selectSql">
        ${alias}.id,${alias}.age,${alias}.name
    </sql>
    <select id="selectUserById" resultType="com.jdbc.pojo.User">
        select 
        <include refid="selectSql">
            <property name="alias" value="u"/>
        </include>
        from User u where u.id = #{userId}
    </select>
```

## 4、结果集映射
### 1、returnType 隐式的使用returnMap
可以使用数据基本类型或者已经创建的实体类，如上代码，如果属性名和select语句中的列名一样mybatis将进行映射到User中。如果属性名和列名不一致可以在sql语句使用别名替代。
```
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```
### 2、returnMap
#### （1）基本用法
该属性更多用在表关联上。但是也可以用来处理属性名和列名不匹配的问题。
```
<resultMap id="BaseUser" type="User">
  <id property="id" column="user_id" />
  <result property="name" column="user_name"/>
  <result property="age" column="user_age"/>
</resultMap>

<select id="selectUserById" resultMap="BaseUser">
	select * from User u where u.id = #{userId}
</select>
```
#### （2）属性
| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| id          | 当前命名空间中的一个唯一标识，用于标识一个结果映射。         |
| type        | 类的完全限定名, 或者一个类型别名（关于内置的类型别名，可以参考上面的表格）。 |
| autoMapping | 如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性autoMappingBehavior。默认值：未设置（unset）。 |
#### （3）子元素
| 元素          | 说明                                 |
| ------------- | ------------------------------------ |
| constructor   | 用于实例化类时将结果注入到构造方法中 |
| id            | 一个ID结果                           |
| result        | 注入字段或者javaBean的结果           |
| association   | 关联（一对一、多对一）               |
| collection    | 集合（一对多）                       |
| discriminator | 通过结果值来决定使用哪个结果映射     |
#### (4)子元素详解
##### 1、<id>,<result>
id 和 result 元素都将一个列的值映射到一个简单数据类型（String, int, double, Date 等）的属性或字段。
这两者之间的唯一不同是，id 元素对应的属性会被标记为对象的标识符，在比较对象实例时使用。 这样可以提高整体的性能，尤其是进行缓存和嵌套结果映射（也就是连接映射）的时候。
###### 属性
| 属性        | 说明                                                     |
| ----------- | -------------------------------------------------------- |
| property    | 因素结果的字段或属性，同javabean实体类的属性名           |
| column      | 对应数据库的列名，同property将javabean和数据库列映射绑定 |
| javaType    | javabean数据类型                                         |
| jdbcType    | 数据流数据类型                                           |
| typeHandler | 类型处理器，支持自定义                                   |

###### 支持的JDBC类型
|          |         |             |               |           |           |
| -------- | ------- | ----------- | ------------- | --------- | --------- |
| BIT      | FLOAT   | CHAR        | TIMESTAMP     | OTHER     | UNDEFINED |
| TINYINT  | REAL    | VARCHAR     | BINARY        | BLOB      | NVARCHAR  |
| SMALLINT | DOUBLE  | LONGVARCHAR | VARBINARY     | CLOB      | NCHAR     |
| INTEGER  | NUMERIC | DATE        | LONGVARBINARY | BOOLEANNC | LOB       |
| BIGINT   | DECIMAL | TIME        | NULL          | CURSOR    | ARRAY     |

##### 2、<association>
用于联表查询，一对一或者多对一的情况使用下面介绍一对一的使用情况，分别有select嵌套查询和嵌套结果映射查询。
###### 嵌套select查询
如果这里association没有使用fetchType=lazy的话，将会出现N+1的问题。这种方式虽然很简单，但在大型数据集或大型数据表上表现不佳。这个问题被称为“N+1 查询问题”。 概括地讲，N+1 查询问题是这样子的：
- 你执行了一个单独的 SQL 语句来获取结果的一个列表（就是“+1”）。
- 对列表返回的每条记录，你执行一个 select 查询语句来为每条记录加载详细信息（就是“N”）。

```
<resultMap id="blogResult" type="com.jdbc.pojo.Blog">
	<id column="id" property="id" jdbcType="BIGINT"></id>
	<result column="title" property="title" jdbcType="VARCHAR"></result>
	<result column="content" property="content" jdbcType="VARCHAR"></result>
	<result column="tag" property="tag" jdbcType="VARCHAR"></result>
	<!-- column 主表外键 -->
	<association property="user" column="user_id" select="selectUser" javaType="com.jdbc.pojo.User" 	fetchType="lazy"/>
</resultMap>

<select id="selectUser" parameterType="int" resultType="com.jdbc.pojo.User"  >
select * from user where id = #{id}
</select>

<select id="selectBlogbyId" resultMap="blogResult">
select * from blog b where b.id = #{blogId}
</select>
```

###### select嵌套查询的association的属性
| 属性      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| column    | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 resultSet.getString(columnName) 方法的参数一样。 |
| select    | 用于加载复杂类型属性的映射语句的 ID，它会从 column 属性指定的列中检索数据，作为参数传递给目标 select 语句。 |
| fetchType | 可选的。有效值为 lazy懒加载（相关数据调用后才进行查询不然是一个代理对象） 和 eager（立即加载） |

###### 关联的嵌套结果映射
所有的关联的嵌套结果映射就是在resultMap进行关联查询，sql语句是联表查询。然后将结果传回到resultMap中进行映射返回。
1、代码示例

```
//User可以重复使用不用重复声明，推荐使用
<resultMap id="BlogWithUser" type="com.jdbc.pojo.Blog">
	<id column="id" property="id" jdbcType="BIGINT"></id>
	<result column="title" property="title" jdbcType="VARCHAR"></result>
	<result column="content" property="content" jdbcType="VARCHAR"></result>
	<result column="tag" property="tag" jdbcType="VARCHAR"></result>
	<association property="user" javaType="com.jdbc.pojo.User" resultMap="UserResult"/>
</resultMap>

<resultMap id="UserResult" type="com.jdbc.pojo.User">
	<id column="id" property="id"></id>
	<result column="name" property="name"></result>
	<result column="age" property="age"></result>
</resultMap>


//User不重复用的情况
<resultMap id="BlogWithUser" type="com.jdbc.pojo.Blog">
	<id column="id" property="id" jdbcType="BIGINT"></id>
	<result column="title" property="title" jdbcType="VARCHAR"></result>
	<result column="content" property="content" jdbcType="VARCHAR"></result>
	<result column="tag" property="tag" jdbcType="VARCHAR"></result>
	<association property="user" javaType="com.jdbc.pojo.User" fetchType="lazy">
		<id column="id" property="id"></id>
		<result column="name" property="name"></result>
		<result column="age" property="age"></result>
	</association>
</resultMap>

<select id="selectBlogbyId" resultMap="BlogWithUser">
	SELECT *
	FROM blog b
	LEFT JOIN USER u
	ON b.user_id = u.id
	WHERE b.id = #{blogId,jdbcType=INTEGER}
</select>
```

##### 3、<collection> 关联集合即一对多的关联
集合元素跟关联元素差不多，有区别的是集合映射的是集合类如ArrayList。下面将把一个用户有多条博客进行关联查询的说明。在User类中对应blog的字段是List<Blog> blogs。
###### select嵌套查询

```
//user.java
public class User implements Serializable {
private Long id;
private String name;
private int age;
private List<Blog> blogs;
//get set
}
//user.xml
<resultMap id="UserResult" type="com.jdbc.pojo.User">
	<collection property="blogs" javaType="ArrayList" ofType="com.jdbc.pojo.Blog" column="id" select="selectByUserId"/> 
</resultMap>
<select id="selectByUserId" resultType="com.jdbc.pojo.Blog">
	select * from blog b where user_id = #{id}
</select>
<select id="selectUserAllBlog" resultMap="UserResult">
SELECT * from User where id = #{userId}
</select>
//userdao.java
User selectUserAllBlog(Integer userId);
```
这里通过selectUserAllBlog接口调用查询用户的博客，查询用户语句后将User数据映射到UserResult中，在collection中property说明映射blog集合的字段，javaType说明是ArrayList集合，ofType说明数据类型是Blog，column将User.id作为参数传给selectByUserId语句进行查询，最后将结果映射到User中。
##### 查询结果如下
```
==>  Preparing: SELECT * from User where id = ? 
==> Parameters: 12(Integer)
<==    Columns: id, name, age
<==        Row: 12, xdp, 12
====>  Preparing: select * from blog b where user_id = ? 
====> Parameters: 12(Integer)
<====    Columns: id, title, content, tag, user_id
<====        Row: 1, mybatis study, study mybatis contnt, mybati, 12
<====        Row: 2, 哈哈哈哈, ok。, 123, 12
<====      Total: 2
<==      Total: 1
```
##### 嵌套结果查询
与上面相似。即使执行selectUserWithBlog语句查询的结果映射到User中。

```xml
<resultMap id="UserResultIn" type="com.jdbc.pojo.User">
<id column="id" property="id" jdbcType="BIGINT"></id>
	<result column="name" property="name" jdbcType="VARCHAR"></result>
	<result column="age" property="age" jdbcType="TINYINT"></result>
	<collection property="blogs" javaType="ArrayList" ofType="com.jdbc.pojo.Blog" resultMap="BlogResult"/> 
</resultMap>

<resultMap id="BlogResult" type="com.jdbc.pojo.Blog">
	<id column="id" property="id" ></id>
	<result column="title" property="title" ></result>
	<result column="content" property="content" ></result>
	<result column="tag" property="tag" ></result>
</resultMap>

<select id="selectUserWithBlog" resultMap="UserResultIn">
	SELECT * FROM USER u
	LEFT OUTER JOIN blog b ON u.id =b.user_id
	WHERE u.id = #{userId};
</select>
```

##### 4、<discriminator> 鉴别器
###### 说明
一个数据库查询可能会返回多个不同的结果集（但总体上还是有一定的联系的）。 鉴别器（discriminator）元素就是被设计来应对这种情况的，另外也能处理其它情况，例如类的继承层次结构。 鉴别器的概念很好理解——它很像 Java 语言中的 switch 语句。
一个鉴别器的定义需要指定 column 和 javaType 属性。column 指定了 MyBatis 查询被比较值的地方。 而 javaType 用来确保使用正确的相等测试（虽然很多情况下字符串的相等测试都可以工作）。例如：
```xml
<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">
    <case value="1" resultMap="carResult"/>
    <case value="2" resultMap="truckResult"/>
    <case value="3" resultMap="vanResult"/>
    <case value="4" resultMap="suvResult"/>
  </discriminator>
</resultMap>
```
MyBatis 会从结果集中得到每条记录，然后比较它的 vehicle type 值。 如果它匹配任意一个鉴别器的 case，就会使用这个 case 指定的结果映射。 这个过程是互斥的.


## 5、cache 缓存
### （1）启动二级缓存,在mapper配置文件添加cache标签
```
<cache/>
```
基本上就是这样。这个简单语句的效果如下:
- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

### (2)缓存属性
```
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

| 属性          | 说明                                    |
| ------------- | --------------------------------------- |
| eviction      | 清除策略，FIFO，LRU（默认），SIFT，WEAK |
| flushInterval | 刷新间隔，默认unset                     |
| size          | 引用数目，默认1024                      |
| readOnly      | 是否只读，false                         |

### (3)自定义缓存
除了上述自定义缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为。
```
<cache type="com.domain.something.MyCustomCache"/>
```
注意需要实现Cache接口。并且提供一个接受String作为id构造器

## 6、cache-ref

# 五、自动映射
当自动映射查询结果时，MyBatis 会获取结果中返回的列名并在 Java 类中查找相同名字的属性（忽略大小写）。 这意味着如果发现了 ID 列和 id 属性，MyBatis 会将列 ID 的值赋给 id 属性。
通常数据库列使用大写字母组成的单词命名，单词间用下划线分隔；而 Java 属性一般遵循**驼峰命名法**约定。为了在这两种命名方式之间启用自动映射，需要将 mapUnderscoreToCamelCase 设置为 true。
甚至在提供了结果映射后，自动映射也能工作。在这种情况下，对于每一个结果映射，在 ResultSet 出现的列，如果没有设置手动映射，将被自动映射。在自动映射处理完毕后，再处理手动映射。 在下面的例子中，id 和 userName 列将被自动映射，hashed_password 列将根据配置进行映射。

```xml
<select id="selectUsers" resultMap="userResultMap">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password
  from some_table
  where id = #{id}
</select>
<resultMap id="userResultMap" type="User">
  <result property="password" column="hashed_password"/>
</resultMap>
```
有三种自动映射等级：
NONE - 禁用自动映射。仅对手动映射的属性进行映射。
PARTIAL - 对除在内部定义了嵌套结果映射（也就是连接的属性）以外的属性进行映射.默认。
FULL - 自动映射所有属性。

# 六、脑图
![图片alt](img/5ec7c5735fcf541e900b6838 ''图片title'')