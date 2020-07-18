## XML 映射器

- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句名为 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的**键是列名**，**值便是结果行中的对应值**。

注意参数符号：

```
#{id}
```



resultType：期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该**设置为集合包含的类型**，而**不是集合本身的类型**。 resultType 和 resultMap 之间只能同时使用一个。

flushCache：将其设置为 true 后，只要语句被调用，都会导致**本地缓存和二级缓存**被清空，默认值：false。

useCache：将其设置为 true 后，将会导致本条语句的**结果被二级缓存缓存起来**，默认值：对 select 元素为 true。



### insert, update 和 delete

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

如果你的数据库还支持多行插入, 你也可以传入一个 `Author` 数组或集合，并返回自动生成的主键。

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```