# 注解

```xml
<!-- 在配置文件中 关联包下的 接口类-->
<mappers>
    <mapper class="dao.DepartmentMapper"/>
</mappers>
```

```java
@Insert("insert into dept values(NULL, #{name}, #{location})")
@Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "deptno")
public int insert(Department dept);
```

```java
public interface DepartmentMapper {

    @Select("select * from dept where deptno=#{id}")
    public Department selectByPK(int id);

    @Select("select * from dept")
    public List<Department> select();
}
```

```java
@Select("select * from dept where deptno=#{id}")
@Results(id = "department", value = {
        @Result(property = "id", column = "deptno"),
        @Result(property = "name", column = "deptno"),
        @Result(property = "location", column = "loc")
})
public Department selectByPK(int id);

@Select("select * from dept")
@ResultMap("department")
public List<Department> select();
```


```java
@Select("select * from emp where empno=#{id}")
@Results(id = "employee", value = {
        @Result(property = "empno", column = "empno"),
        @Result(property = "ename", column = "ename"),
        @Result(property = "job", column = "job"),
        @Result(property = "mgr", column = "mgr"),
        @Result(property = "hiredate", column = "hiredate"),
        @Result(property = "sal", column = "sal"),
        @Result(property = "comm", column = "comm"),
        @Result(property = "dept", column = "deptno", one = @One(select = "dao.DepartmentMapper.selectByPK"))
})
public Employee selectByPK(int id);

@Select("select * from emp where deptno = #{id}")
@ResultMap("employee")
public List<Employee> selectByEmployeeID(int deptno);
```

```java
@Select("select * from dept where deptno=#{id}")
@Results(id = "department", value = {
        @Result(property = "id", column = "deptno"),
        @Result(property = "name", column = "deptno"),
        @Result(property = "location", column = "loc"),
        @Result(property = "employeeList", column = "deptno", many = @Many(select = "dao.EmployeeMapper.selectByDepartmentID"))
})
public Department selectByPK(int id);
```


## <font color="#0088dd">常用功能注解</font>

| 注解 | 目标 | 相对应的 XML | 描述 |
| :-  | :-   | :- | :- |
| `@Param` | Parameter | N/A | 如果你的映射器的方法需要多个参数, 这个注解可以被应用于映射器的方法参数来给每个参数一个名字。否则,多参数将会以它们的顺序位置来被命名 (不包括任何 RowBounds 参数) 比如。 `#{param1}` , `#{param2}` 等 , 这是默认的。使用 `@Param(“person”)`,参数应该被命名为 `#{person}`。|
| `@Insert` <br> `@Update` <br> `@Delete` <br> `@Select` | 方法 | `<insert>` <br> `<update>` <br> `<delete>` <br> `<select>` | 这些注解中的每一个代表了执行的真实 SQL。它们每一个都使用字符串数组 (或单独的字符串)。如果传递的是字符串数组, 它们由每个分隔它们的单独空间串联起来。这就当用 Java 代码构建 SQL 时避免了“丢失空间”的问题。然而,如果你喜欢,也欢迎你串联单独的字符串。`属性:value`，这是字符串数组用来组成单独的 SQL 语句。|
| `@Results` | 方法 | `<resultMap>` | 结果映射的列表, 包含了一个特别结果列如何被映射到属性或字段的详情。属性:value, id。value 属性是 Result 注解的数组。这个id的属性是结果映射的名称。|
| `@Result` |  N/A | `<result>` <br> `<id>` | 在列和属性或字段之间的单独结果映射。属性:id,column, property, javaType ,jdbcType ,type Handler, one,many。id 属性是一个布尔值,表示了应该被用于比较(和在 XML 映射中的 `<id>` 相似)的属性。one 属性是单独的联系, 和 `<association>` 相似 , 而 many 属性是对集合而言的 , 和 `<collection>` 相似。它们这样命名是为了避免名称冲突。|
| `@ResultMap` | 方法 | N/A | 这个注解给@Select或者@SelectProvider提供在XML映射中的 `<resultMap>` 的id。这使得注解的select可以复用那些定义在XML中的ResultMap。如果同一select注解中还存在@Results或者@ConstructorArgs，那么这两个注解将被此注解覆盖。|
| `@One` | N/A | `<association>` | 复杂类型的单独属性值映射。属性: select,已映射语句(也就是映射器方法)的完全限定名,它可以加载合适类型的实例。注意:联合映射在注解 API 中是不支持的。这是因为 Java 注解的限制,不允许循环引用。 fetchType会覆盖全局的配置参数lazyLoadingEnabled。|
| `@Many` | N/A | `<collection>` | 映射到复杂类型的集合属性。属性：select，已映射语句(也就是映射器方法)的全限定名，它可以加载合适类型的实例的集合，fetchType会覆盖全局的配置参数lazyLoadingEnabled。 注意联合映射在注解 API中是不支持的。这是因为 Java 注解的限制,不允许循环引用 |


## 其他注解

| 注解 | 目标 | 相对应的 XML | 描述 |
| :-  | :-   | :- | :- |
| `@InsertProvider` <br> `@UpdateProvider` <br> `@DeleteProvider` <br> `@SelectProvider` | 方法 | `<insert>` <br> `<update>` <br> `<delete>` <br> `<select>` | 这些可选的 SQL 注解允许你指定一个类名和一个方法在执行时来返回运行允许创建动态的 SQL。基于执行的映射语句, MyBatis 会实例化这个类,然后执行由 provider 指定的方法. You can pass objects that passed to arguments of a mapper method, "Mapper interface type" and "Mapper method" via theProviderContext(available since MyBatis 3.4.5 or later) as method argument. (In MyBatis 3.4 or later, it's allow multiple parameters) 属性: type,method。type 属性是类。method 属性是方法名。注意: 这节之后是对类的讨论,它可以帮助你以干净,容于阅读的方式来构建动态 SQL。|
| `@CacheNamespace` | 类 | `<cache>` | 为给定的命名空间 (比如类) 配置缓存。属性:implemetation,eviction, flushInterval,size,readWrite,blocking 和 properties。|
| `@Property` | N/A | `<property>` | Specifies the property value or placeholder(can replace by configuration properties that defined at themybatis-config.xml). Attributes: name, value. (Available on MyBatis 3.4.2+) |
| `@CacheNamespaceRef` | 类 | `<cacheRef>` | 参照另外一个命名空间的缓存来使用。属性:value, name。 If you use this annotation, you should be specified either value or name attribute. For the value attribute specify a java type indicating the namespace(the namespace name become a FQCN of specified java type), and for the name attribute(this attribute is available since 3.4.2) specify a name indicating the namespace.|
| `@ConstructorArgs` | 方法 | `<constructor>` | 收集一组结果传递给一个劫夺对象的构造方法。属性:value,是形式参数的数组。|
| `@Arg` | N/A | `<arg>` <br> `<idArg>` | 单独的构造方法参数 , 是 ConstructorArgs 集合的一部分。属性: id,column,javaType,typeHandler。 id 属性是布尔值, 来标识用于比较的属性,和<idArg>XML 元素相似。|
| `@TypeDiscriminator` | 方法 | `<discriminator>` | 一组实例值被用来决定结果映射的表现。属性: column, javaType, jdbcType, typeHandler,cases。cases 属性就是实例的数组。|
| `@Case` | N/A | `<case>` | 单独实例的值和它对应的映射。属性: value,type,results。Results 属性是结果数组,因此这个注解和实际的 ResultMap 很相似,由下面的 Results 注解指定。|
| `@MapKey` | 方法 | N/A | 复杂类型的集合属性映射。属性 : select,是映射语句(也就是映射器方法)的完全限定名,它可以加载合适类型的一组实例。注意:联合映射在 Java 注解中是不支持的。这是因为 Java 注解的限制,不允许循环引用。|
| `@Options` | 方法 | 映射语句的属性 | 这个注解提供访问交换和配置选项的宽广范围, 它们通常在映射语句上作为属性出现。而不是将每条语句注解变复杂,Options 注解提供连贯清晰的方式来访问它们。属性:useCache=true , flushCache=FlushCachePolicy.DEFAULT , resultSetType=FORWARD_ONLY , statementType=PREPARED , fetchSize=-1 , , timeout=-1 useGeneratedKeys=false , keyProperty=”id” , keyColumn=”” , resultSets=””。理解 Java 注解是很重要的,因为没有办法来指定“null” 作为值。因此,一旦你使用了 Options 注解,语句就受所有默认值的支配。要注意什么样的默认值来避免不期望的行为。|
| `@SelectKey` | 方法 | `<selectKey>` | 该注解复制了 `<selectKey>` 的功能，用在注解了@Insert, @InsertProvider, @Update or@UpdateProvider的方法上。在其他方法上将被忽略。如果你指定了一个@SelectKey注解，然后Mybatis将忽略任何生成的key属性通过设置@Options，或者配置属性。属性： statement是要执行的sql语句的字符串数组，keyProperty是需要更新为新值的参数对象属性， before可以是true或者false分别代表sql语句应该在执行insert之前或者之后， resultType是keyProperty的Java类型， statementType是语句的类型，取Statement, PreparedStatement和CallableStatement对应的STATEMENT, PREPARED或者CALLABLE其中一个，默认是PREPARED。|
| `@ResultType` | Method| N/A | 当使用结果处理器时启用此注解。这种情况下，返回类型为void，所以Mybatis必须有一种方式决定对象的类型，用于构造每行数据。如果有XML的结果映射，使用@ResultMap注解。如果结果类型在XML的`<select>`节点中指定了，就不需要其他的注解了。其他情况下则使用此注解。比如，如果@Select注解在一个方法上将使用结果处理器，返回类型必须是void并且这个注解（或者@ResultMap）是必须的。这个注解将被忽略除非返回类型是void。 | 
| `@Flush` | 方法 | N/A | 如果这个注解使用了，它将调用定义在Mapper接口中的SqlSession#flushStatements()方法。（Mybatis 3.3或者以上）|
