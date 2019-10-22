# 使用 Spring 配合 JUnit 进行单元测试

Spring 集成 junit 进行单元测试，总结一下几种基本的用法：

## 1.直接对 Spring 中注入的 Bean 进行测试（以 DAO 为例）：

在测试类上添加 `@RunWith` 注解指定使用 `SpringJunit` 的测试运行器，`@ContextConfiguration` 注解指定测试用的 Spring 配置文件的位置

> spring-test 和 Junit4 的关系，类似于 slf4j 和 log4j2 的关系。spring-test 并不是『真正干活』的那个，它『调用』的 Junit4 才是在背后默默干活的那个。spring-test 除了可以利用 Junit4 实现单元测试，它还可以利用其它测试框架，比如 TestNG 。

之后我们就可以注入我们需要测试的 bean 进行测试，Junit 在运行测试之前会先解析 spring 的配置文件,初始化 spring 中配置的 bean 。

```java

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {
    "classpath:spring/spring-dao.xml"
})
public class AppTest {

    private static final Logger log = LoggerFactory.getLogger(AppTest.class);

    @Autowired
    private DepartmentDAO departmentDao;

    @Test
    public void demo1() {
        Department dept = departmentDao.selectByPrimaryKey(40);
        log.info("{}", dept);
    }
}
```

## 2. spring-test 利用事务避免污染数据库

如果涉及测试增删改的 DAO 方法，或者是测试涉及这些 DAO 的 Service 方法，每一个 Test 方法执行结束后都会对数据库造成影响，从而极大可能影响后续 Test 方法的执行。（因为对于第二个 Test 方法而言，数据库环境发生了变化，初始条件可能就已经不满足了）。

为此，spring-test 利用事务的回滚可以在 Test 方法执行结束后，撤销 Test 方法对数据库造成的影响。注意，你的 spring 配置文件中，必须有相关的事务配置部分，即，必须引入 `spring-service.xml` 配置文件

```java
@Test
@Transactional
@Rollback
public void demo2() {
    Department dept = departmentDao.selectByPrimaryKey(40);
    log.info("{}", dept);

    dept.setDname(dept.getDname() + ".");
    departmentDao.updateByPrimaryKey(dept);

    dept = departmentDao.selectByPrimaryKey(40);
    log.info("{}", dept);
}
```

- `@Transactional` 表示启用事务。
- `@Rollback` 表示本方法执行结束后事务回滚。


`@Transactional` 和 `@Rollback` 还可以直接标注在类上，相当于对测试类下的所有的测试方法进行了统一设置。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {
    "classpath:spring/spring-dao.xml", 
    "classpath:spring/spring-service.xml"
})
@Transactional
@Rollback
public class AppTest {
    ...
}
```

## 3. 使用 H2 内存数据库

我们在测试时，对数据库会有如下要求：

- 单元测试必须执行隔离的代码。因为绝大多数 TestCase 都是独立的，各个 TestCase 之间不能相互影响。
- 单元测试必须快速执行。因为一个方法不仅会有一个 TestCase，理论上测试代码要远多于业务代码。

以上两个难点决定了嵌入式数据库（H2、HSQLDB、Derby、Java DB 等）的使用价值。嵌入式数据库使用场景较少，但是是配合 JUnit 进行代码测试确实极佳的选择。

H2 是一个由 Java 代码实现的内嵌式数据库，它支持多种运行模式（其中我们只关心/使用`内存运行模式`）。相较于它的竞争者而言，它最大的特点在于，它兼容 MySQL，虽然仍有些不完全一致的地方，但相较而言那都是些细枝末节，无关紧要之处。

在 pom.xml 中添加 h2database 的依赖。如果是在非Maven项目中使用，则下载该 h2 的 jar 包并加入项目的classpath 中。

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
    <scope>test</scope> <!-- 由于我们仅用它来进行 JUnit 测试，
                             因此只在测试环境中使用它。
                             项目发布时，项目的包中不需要包含它 -->
</dependency>
```

```java
private static final String DRIVER = "org.h2.Driver";
private static final String URL = "jdbc:h2:mem:scott;MODE=MYSQL;DB_CLOSE_DELAY=-1";
private static final String USERNAME = "sa";
private static final String PASSWORD = "";
```

- **jdbc:h2:mem:scott**
  
  > - 这是数据库 URL 的核心部分，其中 `mem` 就表示使用内存模式的 H2。H2 的各种不同的使用/运行模式，主要体现在这个部分。
  > - 此处指令连接 scott 数据库，h2 就会自动帮我们创建 scott 数据库，因此后续的 sql 语句中无须再指定创建 scott 数据库，也不用再使用 `use scott` 切换到数据库。

- **MODE=MYSQL**

  > - H2 并不是唯一的嵌入式数据库，也不是唯一具有内存模式的嵌入式数据库，但是它是与 MySQL 语法最兼容的具有内存模式的嵌入式数据库（虽然仍有些许特殊区别），这也是 JUnit中 首选 H2 的原因。

- **DB_CLOSE_DELAY=-1**

  > - 默认情况下，H2 内存中的数据库是在最后一个连接断开后关闭数据库，即删除数据库及其中所有数据。
  > - 设置为 `-1` 表示不以连接数作为判断标准，而是持续保持数据库的存在，直到程序运行结束。

- **用户名和密码**
  > 由于使用的是 h2 的内存模式，所以这里并不存在实际上的连接校验身份的功能。因此用户名密码并没有实际的作用。按惯例写成 `"sa"` 和 `""` 既可。 


H2 有多种使用模式，在此我们只关心它的 *内存模式* 。即，将 database 和 table，建立在内存中。H2 数据库不需要专门的去启动/运行它，直接连接即可！

在 spring-dao.xml 其实很简单，只需要修改数据库连接池的四大连接属性即可。

```xml
<!-- 数据库连接池  -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" 
        init-method="init" destroy-method="close">
    <property name="driverClassName" value="org.h2.Driver"/>
    <property name="url" value="jdbc:h2:mem:scott;MODE=MYSQL;DB_CLOSE_DELAY=-1"/>
    <property name="username" value="sa"/>
    <property name="password" value=""/>
</bean>
```

初始化数据库可以借助于 spring-dao.xml 中 `<jdbc:initialize-database>` 配置：

```xml
<jdbc:initialize-database data-source="dataSource"  ignore-failures="DROPS">  
    <jdbc:script location="classpath:sql/schema.sql" />  
    <jdbc:script location="classpath:sql/import-data.sql" encoding="UTF-8" />  
</jdbc:initialize-database>  
```

`jdbc:initialize-database` 这个标签的作用是在工程启动时，去执行一些 sql，也就是初始化数据库。比如向数据库中建立一些表及插入一些初始数据等。这些 sql 的路径需要在其子标签 `jdbc:script` 中去指定。

`jdbc:initialize-database` 标签

> - `dataSource` 属性，引一个配置好的数据库连接池。
> - `ignore-failures` 有三个值：`NONE`、`DROPS`、`ALL`。
>   - 设置为 `NONE` 时，表示不忽略任何错误，也就是说，当 sql 执行报错时服务启动终止；
>   - 设置为 `DROPS` 时，忽略删除错误，如当 sql 中有一个删除表 `drop table xxx` 的 sql，而 xxx 表不存在，此时这个错误会被忽略；
>   - 设置为 `ALL` 时，忽略任何错误。


## 2. 对 Spring MVC 进行测试

Spring 3.2 之后出现了 `org.springframework.test.web.servlet.MockMvc` 类,对 Spring MVC 单元测试进行支持。

```java
package com.xja.test;

import com.jiaoyiping.baseproject.privilege.controller.MeunController;
import com.jiaoyiping.baseproject.training.bean.Person;
import junit.framework.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.servlet.ModelAndView;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations={
    "classpath:spring/spring-*.xml" 
}) 
public class TestMockMvc {

    @Autowired
    private org.springframework.web.context.WebApplicationContext context;

    MockMvc mockMvc;

    @Before
    public void before() {
        mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }

    @Test
    public void demo() {
        String result = mockMvc.perform(
                get("/dept.do")
                .param("deptno", "10"))
            .andReturn()
            .getResponse()
            .getContentAsString();

        System.out.println(result);
    }

    @Test
    public void testGetDepartment() throws Exception {
        ResultActions actions = mockMvc.perform(
                get("/dept.do")
                .param("deptno", "10")
        ).andExpect(status().isOk());
           
        String viewName = actions.andReturn().getModelAndView().getViewName(); 
        Map<String, Object> model = actions.andReturn().getModelAndView().getModel();
       
        Department dept = (Department) model.get("xxx");

        log.info("{} {}", viewName,dept);
           
        Assert.assertEquals(new Integer(10), dept.getDeptno());
    }

    @Test
    public void testUpdateDepartment() throws Exception {
        ResultActions actions = mockMvc.perform(
            post("/dept_update.do")
            .param("deptno", "40")
            .param("dname", "Testing")
            .param("loc", "BeiJing")
        ).andExpect(status().isOk());
            
        String viewName = actions.andReturn().getModelAndView().getViewName(); 

        log.info("{}", viewName);
            
        Assert.assertEquals("hello", viewName);
    
        Department dept = departmentDAO.selectByPrimaryKey(40);

        Assert.assertEquals("Testing", dept.getDname());
        Assert.assertEquals("BeiJing", dept.getLoc());
    }
}
```