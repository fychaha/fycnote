## 1. 基本概念

Junit 是 java 领域占有率非常高的一个单元测试框架，已经成为了单元测试的标准。

单元测试的编写原则:

1. 在 eclipse 中创建 1 个 source folder 命名为 test（使用 Maven 后已要求创建）
2. 测试类所在的包要求和被测试类的包一致
3. 测试类要使 Test 作为开头或结，如 UserServiceTest
4. 测试类的每个方法，都必须是可以独立执行的，不存在顺序或依赖

```java
package com.hemiao; 

import org.junit.After; 
import org.junit.AfterClass; 
import org.junit.Before;
import org.junit.BeforeClass;  
import org.junit.Test;  

/* 
 * 测试类最基本的格式  
 */
public class TestUser {  

    @BeforeClass
    public static void init() {  
        /*  
         * 此方法只会在运行所有单元测试前执行一次,   
         * 通常用来获取数据库连接, Spring 管理的 Bean 等等  
         */
    }  

    @Before
    public void setUp() {  
        /*  
         * 此方法运行每个单元测试前都会执行,   
         * 通常用来准备数据或获取单元测试依赖的数据或对象  
         */
    }  

    @Test
    public void testAddUser() {  
        /*  
         * 测试类的主要方法,在这里写所有的测试业务逻辑  
         */
    }  

    @Test(expected = Exception.class)  
    public void testDleteUser() {  
        /*  
         * 请注意注解上的experced,使用该参数代表可以认为   
         * 这个单元测试方法会抛出Exception的异常,若然没有视为不通过  
         */
    }  

    @Test(timeout = 1000)  
    public void testUpdateUser() {  
        /*  
         * 请注意注解上的timeout，使用该参数代表
         * 该单元测试需要在1000毫秒内完成，否则视为不通过
         * 可以用做简单的性能测试  
         */
    }  

    @After
    public void tearDown() {  
        /*  
         * 此方法运行每个单元测试后都会执行,   
         * 主要用来和setUp对应,清理获取的资源  
         */
    }  

    @AfterClass
    public static void destroy() {  
        /*  
         * 此方法会在运行所有单元测试后执行一次,   
         * 通常用来释放资源,例如数据库连接,IO流等等  
         */
    }  
}  
```

当你需要测试一个『应该』抛出异常的逻辑时，可以使用 `@Test` 注解的 `expected` 属性。

```java
@Test(expected = IndexOutOfBoundsException.class)
public void testIndexOutOfBoundsException() {
    List<String> list = new ArrayList<>();
    String str = list.get(0);}
}
```

---

当我们定义了多个测试方法,但还没提供实现时,可以选择用以下方式来提醒或保证还存在没实现的方法。

```java
public class TestUser {  

    @Test
    @Ignore
    public void testAddUser() {  
        //添加@Ignore后該方法將被忽略而不去执行  
    }  

    @Test
    public void testGetUser() {  
        //当运行测试时会提示该方法还没实现  
        fail("该方法暂时没提供实现");  
    }  
}  
```

---

在运行单元测试的时候都是一个单元测试类这样来运行的,但项目当中可能存在上百个测试类的时候,每次要手动一个一个单元测试的运行简直就是噩梦。通过创建 `Junit Test Suite` 可实现批量执行测试用例。

该类会生成以下的代码

```java
@RunWith(Suite.class)  
@SuiteClasses({ TestUser.class })  
public class AllTests {  
    /*  
     * @RunWith 代表是运行 Suite 的类,固定写法  
     * @SuiteClasses 中接受一个数组，里面为你需要  
     * 集中运行的测试类的class以上代码为自动生成  
     * 当然除了这个办法外，你也可以通过 Maven 来运行单元测试,这里不过多叙述  
     */
}  
```