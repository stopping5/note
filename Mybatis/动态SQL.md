

# 一、IF

if标签在Where关键查询中使用，如下：

如果传入的参数title不为null，则添加AND title like #{title} sql语句执行模糊查询。

```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```



# 二、Choose

**搭配标签choose,when,otherwise**

choose相当于java中的switch语句，when相当于case，otherwise相当于default。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
     //上方when没有出现执行
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```



# 三、动态条件查询-where、trim、set

条件查询中使用IF语句，如果state为null的话，后面拼接的sql语句将会是 SELECT * FROM BLOG  WHERE  **AND ** title like #{title}；导致查询失败。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE
  <if test="state != null">
    state = #{state}
  </if>
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>

```

## where

*where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

## trim

用于where不能满足我们需求的时候使用。

```xml
<select id="findUserInfoByTrim" parameterType="Map"
		resultMap="UserInfoResult">
		select * from userinfo
		<trim prefix="where" prefixOverrides="and|or">
			<if test="department!=null">
				AND department like #{department}
			</if>
			<if test="gender!=null">
				AND gender=#{gender}
			</if>
			<if test="position!=null">
				AND position like #{position}
			</if>
		</trim>
</select>
```

##  set



# 四、Foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在**构建 IN 条件语句**的时候）。Foreach用于动态的构建IN条件语句比如：

```xml
sql构建IN条件：
SELECT
	* 
FROM
	`facilities` 
WHERE
	num IN ( 10, 90, 100 )

<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

*foreach* 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。这个元素也不会错误地添加多余的分隔符，看它多智能！

>**提示** 你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 *foreach*。**List**当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。当使用 **Map **对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

至此，我们已经完成了与 XML 配置及映射文件相关的讨论。下一章将详细探讨 Java API，以便你能充分利用已经创建的映射配置。







