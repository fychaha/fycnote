# 注解开发 mybatis 的 mapper 属性和字段不对应问题

Mybatis给我们提供了一种映射方式，如果属性的命名是遵从驼峰命名法的，数据列名遵从下划线命名。MyBatis支持使用注解来配置映射语句，不再需要在XML配置文件中配置。
学习内容

- @Results 对应resultMap
- @Result 对应result

这两个注解是应用在方法的级别上的，也就是在mapper方法。

## <font color="#0088dd">@Results 结果映射</font>

Mybatis 不能通过 @Column 注解或者直接使用实体类的属性名作为数据列名，而是需要自己指定实体类属性和数据表中列名之间的映射关系

## 解决方案

我们可以将查询结果通过别名与JavaBean属性映射起来。当然使用@Results注解也可以将指定列与指定JavaBean属性映射起来
执行SELECT查询的：

```java
@Select("SELECT * FROM `dept`")
@Results({
    @Result(property = "departmentNumber", column = "deptno"),
    @Result(property = "departmentName", column = "dname"),
    @Result(property = "location",column = "loc"),
})
public List<Department> queryAll();

@Select("SELECT * FROM `dept` WHERE `deptno` = #{id}")
@Results({
    @Result(property = "departmentNumber", column = "deptno"),
    @Result(property = "departmentName", column = "dname"),
    @Result(property = "location",column = "loc"),
})
public WxMessageConfig queryByPK(int id);
```

> 另一种方案是：通过使用在SQL语句中定义别名完成映射。例如：<br>
@Select("SELECT <br>deptno AS departmentNumber, <br>dname AS departmentName, <br>loc AS location <br> FROM dept WHERE deptno = #{id}")


由于注解是针对方法的，对于Mapper中的每个操作数据库的方法都必须有相同的注解完成映射关系的建立，导致很多是重复的。

> ## 使用 @RresultMap 共用 @Results

为了解决重复使用，那就要让他变成一个有id的整体，其他地方要用就直接调用

```java
@ResultMap(“<id>”)
```

@Result中 通过 id 属性引用这个 resultMap 

```java
@Select("SELECT deptno, dname, loc FROM dept WHERE deptno = #{id}")
@Results(id = "deptMapper", value = { 
    @Result(column = "deptno", property = "departmentNumber"),
    @Result(column = "dname", property = "departmentName"),
    @Result(column = "loc", property = "location")
}) 
public Department findByPK(int id); 

@Select("SELECT * FROM dept") 
@ResultMap("deptMapper") 
public List<Department> fingAll();
```



