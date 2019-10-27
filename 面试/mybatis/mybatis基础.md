# Mybatis

### 简单介绍mybatis

mybatis是一个持久层的半对象关系映射框架，它对jdbc操作数据库的过程进行了封装，是我们能够只需要专注与sql语句本身，不用考虑驱动，创建连接等。mybatis通过xml配置文件或者注解，通过java对象和要执行的statement中的SQL映射成最终的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回给我们。

### mybatis层次

- api接口层：提供给外部使用的接口api
- 数据处理层：负责具体的sql
- 基础支撑层：连接管理，事务管理，配置加载和缓存处理。

### $和#的区别

1. #把传入的参数当做空字符串处理，$直接显示
2. #能防止sql注入，底层用的是preparestatement，而$底层用statement。
3. 能用#就用#

### 说说mybatis的一级缓存与二级缓存

- 一级缓存：一级缓存作用域是sqlsession级别的，在同一个sqlsession中执行两条相同的查询语句，第一次会将查询到的数据存储到缓存中，第二次就直接从缓存里查找，这样可以提高效率，当一个sqlsession结束后，一级缓存也将消失。mybatis默认开启一级缓存。
- 二级缓存：二级缓存是mapper级别的，作用域是同一个namespace，执行两次查找语句，第一次结果放缓存，第二次去缓存查找。默认不开启。

### 说说mybatis的编程步骤

1. 创建SqlSessionFactoryBuilder
2. 通过SqlsessionFactoryBuilder创建sqlSessionFactory
3. 通过SqlSessionFactorry创建sqlSession
4. 通过sqlSession连接数据库
5. 调用session.commit()提交事务
6. 调用session.close()关闭连接

### 怎么设置主键回填？

- keyProperty：指定需要回填的bean属性（对应数据库主键列）。

- useGeneratedKeys：启用mybatis主键数据库维护，并使用主键回填功能。

  ````xml
  <insert id="insert" parameterType="com.fyc.Student" useGeneratedKeys="ture" keyProperty="id">
    INSERT INTO student(name, loc) VALUES(#{name}, #{loc});
  </insert>
  ````

  

### 说说mybatis的延迟加载

如果一个对象a关联另一个对象b，那么在查询a的时候，会去关联查询b。把b的全部信息拿出了又没必要。所以何时加载b对象就是程序员关心的话题。

b对象的加载分三种时机：

- 立即加载（无脑查就完事了）
- 激进地延迟加载：（）
- 延迟加载：

通过设置启用延迟加载

````xml
<settings>
  <setting name="lazyLoadingEnabled" value="true"/>
</settings>
````

启用延迟加载后，mybatis又是默认激进地延迟加载。

激进延迟加载根据mybatis的内部判断规则，有时等同于立即加载，有时等同于普通加载。

需要真正地延迟加载，要这样配置：

`````xml
<settings>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="aggressiveLazyLoading" value="false"/>
</settings>
`````

这样，在查询a对象时，当你真正用到b对象的属性时，才会去加载查询b对象。