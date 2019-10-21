---
title: SpringData JPA总结
tags: SpringData
categories: SpringData
---

# 基本使用

### 1，添加依赖

```xml
<dependency>
    <groupId>org.Springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 2，配置文件

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    # 日志中显示 SQL 语句
    show-sql: true
    properties:
      hibernate:
        # 指定生成表的存储引擎为 InneoDB
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
        format_sql: true
        hbm2ddl:
          auto: update
```

### 3，修改实体类

- `@Entity(name="EntityName")` ***必须***

  > 用来标注一个数据库对应的实体，数据库中创建的表名默认和类名一致。其中，name 为可选,对应数据库中一个表，使用此注解标记 Pojo 是一个 JPA 实体。

- `@Table(name="",catalog="",schema="")` *可选*

  > 用来标注一个数据库对应的实体，数据库中创建的表名默认和类名一致。通常和 `@Entity` 配合使用，只能标注在实体的 class 定义处，表示实体对应的数据库表的信息。

  DBMS-catalog-schema-table 是四级概念，但不是所有的数据库系统都支持这四级。MySql 就不支持其中的 catalog ，而 schema 就是 mysql 中的 database 。

- `@Id` ***必须***

  > @Id 定义了映射到数据库表的主键的属性，一个实体只能有一个属性被映射为主键。

- `@GeneratedValue(strategy=GenerationType, generator="")` *可选*

  > strategy: 表示主键生成策略,有 `AUTO`、`INDENTITY`、`SEQUENCE` 和 `TABLE` 4 种。generator：表示主键生成器的名称。

- `@Column(name="user_code", nullable=false, length=32)` *可选*

  > @Column 描述了数据库表中该字段的详细定义,这对于根据 JPA 注解生成数据库表结构的工具。
  >
  > name: 表示数据库表中该字段的名称,默认情形属性名称一致;
  >
  > nullable: 表示该字段是否允许为 null,默认为 true;
  >
  > unique: 表示该字段是否是唯一标识,默认为 false;
  >
  > length: 表示该字段的大小,仅对 String 类型的字段有效。

- `@Transient` *可选*

  > @Transient 表示该属性并非一个到数据库表的字段的映射,ORM 框架将忽略该属性。

- `@Enumerated` *可选*

  > 使用枚举的时候，我们希望数据库中存储的是枚举对应的 String 类型，而不是枚举的索引值，需要在属性上面添加 `@Enumerated(EnumType.STRING)` 注解。

```java
@Entity
@Table(name="表名", schema ="数据库名")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, unique = true)
    private String userName;

    @Column(nullable = false)
    private String passWord;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = true, unique = true)
    private String nickName;

    @Column(nullable = false)
    private String regTime;

	// getter / settet ...
}
```

### 4，构建对应的Repository

1. 继承JpaRepository<对应实体类,主键的类型>，这样就默认生成了很多的增删改的方法

2. 还可以使用关键字自定义方法

   **具体的关键字，使用方法和生产成 SQL 如下表所示**

   | Keyword            | Sample                                   | JPQL snippet                                                 |
   | :----------------- | :--------------------------------------- | :----------------------------------------------------------- |
   | And                | findByLastnameAndFirstname               | … where x.lastname = ?1 and x.firstname  = ?2                |
   | Or                 | findByLastnameOrFirstname                | … where x.lastname = ?1 or x.firstname = ?2                  |
   | Is, Equals         | findByFirstnameIs, findByFirstnameEquals | … where x.firstname = ?1                                     |
   | Between            | findByStartDateBetween                   | … where x.startDate between ?1 and ?2                        |
   | LessThan           | findByAgeLessThan                        | … where x.age < ?1                                           |
   | LessThanEqual      | findByAgeLessThanEqual                   | … where x.age <= ?1                                          |
   | GreaterThan        | findByAgeGreaterThan                     | … where x.age > ?1                                           |
   | GreaterThanEqual   | findByAgeGreaterThanEqual                | … where x.age >= ?1                                          |
   | After              | findByStartDateAfter                     | … where x.startDate > ?1                                     |
   | Before             | findByStartDateBefore                    | … where x.startDate < ?1                                     |
   | IsNull             | findByAgeIsNull                          | … where x.age is null                                        |
   | IsNotNull, NotNull | findByAge(Is)NotNull                     | … where x.age not null                                       |
   | Like               | findByFirstnameLike                      | … where x.firstname like ?1                                  |
   | NotLike            | findByFirstnameNotLike                   | … where x.firstname not like ?1                              |
   | StartingWith       | findByFirstnameStartingWith              | … where x.firstname like ?1 (parameter bound with appended %) |
   | EndingWith         | findByFirstnameEndingWith                | … where x.firstname like ?1 (parameter bound with prepended %) |
   | Containing         | findByFirstnameContaining                | … where x.firstname like ?1 (parameter bound wrapped in %)   |
   | OrderBy            | findByAgeOrderByLastnameDesc             | … where x.age = ?1 order by x.lastname desc                  |
   | Not                | findByLastnameNot                        | … where x.lastname <> ?1                                     |
   | In                 | findByAgeIn(Collection ages)             | … where x.age in ?1                                          |
   | NotIn              | findByAgeNotIn(Collection age)           | … where x.age not in ?1                                      |
   | TRUE               | findByActiveTrue()                       | … where x.active = true                                      |
   | FALSE              | findByActiveFalse()                      | … where x.active = false                                     |
   | IgnoreCase         | findByFirstnameIgnoreCase                | … where UPPER(x.firstame) = UPPER(?1)                        |

### 5，使用构建好的repository对象

直接调用其中的方法即可



# 高级使用

## 分页查询

JPA内置了分页查询，使用方法如下

1. 创建`Pageable`对象

   ```java
   Pageable pageable=PageRequest.of(当前页,每页显示条数)
   ```

2. 调用查询方法，在方法参数的最后传入上面创建的Pageable对象，返回值是一个Page类型

   当然也可以是一个Slice类型，Page继承自Slice

   Page 和 Slice 的区别如下:

   - Page 接口继承自 Slice 接口，而 Slice 继承自 Iterable 接口。
   - Page 接口扩展了 Slice 接口，添加了获取总页数和元素总数量的方法，因此，返回 Page 接口时，必须执行两条 SQL，一条复杂查询分页数据，另一条负责统计数据数量。
   - 返回 Slice 结果时，查询的 SQL 只会有查询分页数据这一条，不统计数据数量。
   - 用途不一样：Slice 不需要知道总页数、总数据量，只需要知道是否有下一页、上一页，是否是首页、尾页等，比如前端滑动加载一页可用；而 Page 知道总页数、总数据量，可以用于展示具体的页数信息，比如后台分页查询。

3. 使用返回的Page对象获取其中的数据以及封装的其他信息

## 限制查询

也就是只需要固定的条目的数据，相当于变现的条件查询

有时候我们只需要查询前 N 个元素，或者只取前一个实体。

```java
User findFirstByOrderByLastnameAsc();
User findTopByOrderByAgeDesc();
Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);
List<User> findFirst10ByLastname(String lastname, Sort sort);
List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

## 复杂查询

也就是面对很多个条件的查询方法，本质可以用上面的查询方法一点一点的拼接。但是下面使用的方法可以简化，不要求掌握，知道这么回事就可以了

以前总是会使用`Criteria`来执行一些复杂的查询方法，官方封装了`Criteria`提供了一个`JpaSpecificationExecutor`来帮助执行一些复杂的操作

使用方法

1. 对应的Repository实现`JpaSpecificationExecutor<对应实体类>`

   ```java
   例：
   public interface UserDetailRepository 
           extends JpaSpecificationExecutor<UserDetail>, 
                   JpaRepository<UserDetail, Long> {
   }
   ```

2. 在service中

   ```java
    public Page<UserDetail> findByCondition(UserDetailParam detailParam, Pageable pageable) {
           return userDetailRepository.findAll((root, query, cb) -> {
               List<Predicate> predicates = new ArrayList<Predicate>();
       
               // equal 示例：“introduction = xxx”
               if (!StringUtils.isNullOrEmpty(detailParam.getIntroduction())){
                   predicates.add(
                       		cb.equal(
                                   root.get("introduction"),
                                   detailParam.getIntroduction())
                   );
               }
               
               //like 示例：realName like ?
               if (!StringUtils.isNullOrEmpty(detailParam.getRealName())) {
                   predicates.add(
                       cb.like(
                           root.get("realName"),
                           "%" + detailParam.getRealName() + "%")
                   );
               }
               
               //between 示例：age between (?, ?)
               if (detailParam.getMinAge()!=null && detailParam.getMaxAge()!=null) {
                   Predicate agePredicate = cb.between(
                       root.get("age"), 
                       detailParam.getMinAge(), 
                       detailParam.getMaxAge());
                   predicates.add(agePredicate);
               }
               
               // greaterThan 大于等于示例
               if (detailParam.getMinAge()!=null) {
                   predicates.add(
                       cb.greaterThan(
                       	root.get("age"),
                       	detailParam.getMinAge())
                   );
               }
              
               return query.where(
                   predicates.toArray(
                       new Predicate[predicates.size()])
               ).getRestriction();
           }, pageable);
       }
   ```

3. 看不懂算了，需要的时候在使用就是了，反正我也不懂，哈哈哈

## 多表查询

 有两种实现方式

1. 利用 Hibernate 的级联查询来实现
2. 创建一个结果集的接口来接收连表查询后的结果

第二种实现方式

1. 创建一个结果集的接口类

   接口类的内容来源于需要关联的表中的所有数据

   ```java
   public interface UserInfo {
       String getUserName();
       String getEmail();
       String getAddress();
       String getHobby();
   }
   ```

2. 在对应的Repository中创建查询方法

   使用Hql来帮助查询

   HQL使用的是对应的类，所有对应使用类名和属性名，传参数使用的是`?1,?2`这样的形式

   ```java
   @Query("select 
          u.userName as userName, u.email as email, 
          		d.address as address, d.hobby as hobby 
          from User u, UserDetail d 
          where u.id = d.userId 
          and d.hobby = ?1 ")
   List<UserInfo> findUserInfo(String hobby);
   ```

3. 执行这个方法，返回值是`List<接口>`

   ```java
   List<UserInfo> userInfos=userDetailRepository.findUserInfo("钓⻥鱼");
   ```



完结...















​	