## trim

* prefix：给sql语句拼接的**前缀**
* suffix：给sql语句拼接的**后缀**
* prefixOverrides：**去除sql语句前面的关键字或者字符**，假设该属性指定为"AND"，当sql语句的开头为"**AND**"，trim标签将会**去除该"AND"**
* suffixOverrides：**去除sql语句后面的关键字或者字符**

```xml
<trim prefix="WHERE" prefixOverrides="AND">
	<if test="state != null">
	  state = #{state}
	</if> 
	<if test="title != null">
	  AND title like #{title}
	</if>
	<if test="author != null and author.name != null">
	  AND author_name like #{author.name}
	</if>
</trim>
```

跟 where 同作用

```xml
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
```





