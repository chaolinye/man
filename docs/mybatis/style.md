# Mybatis CURD 的三种风格

## 最强大的 XML 映射文件

优点：直接使用 sql 语句，功能最为强大

缺点：需要额外的 xml 文件，缺乏检查

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

## 映射注解


优点：sql 代码和接口放在一处，方便阅读

缺点：功能受限，缺乏检查, sql 较长时可读性差

```java
@Results(id = "userResult", value = {
  @Result(property = "id", column = "uid", id = true),
  @Result(property = "firstName", column = "first_name"),
  @Result(property = "lastName", column = "last_name")
})
@Select("select * from users where id = #{id}")
User getUserById(Integer id);

@Results(id = "companyResults")
@ConstructorArgs({
  @Arg(column = "cid", javaType = Integer.class, id = true),
  @Arg(column = "name", javaType = String.class)
})
@Select("select * from company where id = #{id}")
Company getCompanyById(Integer id);
```

## Mybatis-Plus lambda 写法

```java
userMapper.update(null, Wrappers.<User>lambdaUpdate().set(User::getName, "newName").eq(User:getId, 12))
```

## 建议

优先使用 Mybatis-Plus lambda 写法，复杂查询使用 XML 映射文件

## Reference

- [Mybatis Plus 条件构造器](https://baomidou.com/pages/10c804/#isnull)
