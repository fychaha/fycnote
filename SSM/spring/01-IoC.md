# Spring IoC

```xml
<properties>
    ...
    <spring.version>4.3.2.RELEASE</spring.version>
    ...
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>

    ...

</dependencies>
```

## 一、实例化 Spring IoC 容器。

Spring 核心容器的理论很简单：**Spring 核心容器就是一个超级大工厂，所有的对象都会被当成 Spring 核心容器管理的对象**。

你必须实例化 Spring IoC 容器，读取其配置文件来创建 Bean 实例。然后你可以从 Spring IoC 容器中得到可用的 Bean 实例。

```
BeanFactory
└── ApplicationContext
    └── ClassPathXmlApplicationContext
```

Spring IoC 容器主要是基于 <font color="0088dd">**BeanFactory**</font> 和 <font color="#0088dd">**ApplicationContext**</font> 两个接口：

- **`BeanFactory`** 是 Spring IoC 容器的顶层接口
- **`ApplicationContext`** 是最常用接口
- **`ClassPathXmlApplicationContext`** 是 ApplicationContext 的实现类。顾名思义，它从 classpath 中加载一个 XML 配置文件，构建一个应用程序上下文。你也可以指定多个配置文件。

**BeanFactory** 接口定义了 Spring IoC 整个体系中中最重要的方法（簇）：<font color="0088dd">**getBean（）**</font> 方法。这个方法用于从 Spring IoC 容器中获得 Bean 。


```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
ApplicationContext context = new ClassPathXmlApplicationContext("service.xml", "dao.xml");
```

在获得应用程序上下文（也就是IoC容器）后，你只需要调用 getBean() 方法并传入唯一的 Bean ID/名称，就可以获得容器中的 Bean。

```java
Human tom = context.getBean("tom", Human.class);
```

## 二、Spring 创建 Bean

Spring 可以“帮”你创建、管理 Bean 对象，但是前提是你必须“告诉”它创建、管理哪些 Bean 对象。

Spring 允许你在一个（或多个）XML 配置文件中配置 Bean，对于 Spring IoC 容器，这个配置文件就是创建、管理 Bean 的依据。

一个 Spring XML 配置文件的基本样式是：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">

</beans>
```

创建 Bean 的方式常见三种：

> - 类自身的构造方法
> - 工厂类提供的工厂方法
> - 工厂对象提供的工厂方法

### 1. 使用类自身的构造方法

每个 bean 都必须提供一个唯一的名称或id，以及一个完全限定的类名，用来让 Spring IoC 容器对其进行创建。Spring 通过类的构造方法来创建 Bean 对象。

```xml
<beans ...>
    <bean id="xxx" class="xxx.xxx.Xxx"></bean>
</beans>
```

### 2. 使用工厂类提供的工厂方法

```xml
<bean id="tom" class="工厂类" factory-method="工厂方法">
</bean>
```


### 3. 使用工厂对象提供的工厂方法

```xml
<!-- 声明创建工厂对象 -->
<bean id="factory" class="工厂类"/>

<bean id="tom" factory-bean="工厂对象id" factory-method="工厂方法">
</bean>
```


## 三、Spring 装配简单类型属性

装配，即为 Bean 的属性进行 初始化 / 赋值。

<font color="0088dd">**简单类型**</font> 是指： 基本数据类型、基本数据类型包装类 和 字符串 。

装配方式有两种：

> - 通过 构造方法 装配
> - 通过 setter 装配


### 1. 构造方法装配

通过构造方法装配，即设置 Spring 通过调用有参构造方法（默认是调用无参构造方法）来创建对象。在 bean 元素内使用 <font color="0088dd">**constructor-arg**</font> 子元素，即可触发构造方法装配。

```xml
<bean id="..." class="...">
    <constructor-arg index="x" value="xx"/>
    ...
</bean>
```

<font color="0088dd">**index**</font> 属性表示构造函数形参 **索引**（从0开始）。如果参数的类型具有唯一性，那么可以使用 <font color="0088dd">**type**</font> 属性，通过 **参数类型** 来指定构造方法和参数值。



为了简化配置，Spring 提供了一个名为 <font color="0088dd">**c**</font> 的 schema，来简化配置。

```xml
<beans xmlns="..."
       xmlns:xsi="..."
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="...">

    <bean id="..." class="..." c:_0="xxx" c:_1="xxx" ... />
</beans>
```

使用这种简写方式，完全不用出现 **costructor-arg** 子元素，只需在 **bean** 元素中多增加几个 <font color="0088dd">**c:_索引="参数值"**</font> 这样的属性。


### 2. setter 装配

通过 setter 装配，即设置 Spring 在（通过无参构造方法）创建对象后，通过调用对象的属性的 setter 方法来为对象的属性赋值。在 bean 元素内使用 <font color="0088dd">**property**</font> 子元素，即可触发 setter 装配。

```xml
<bean id="..." class="...">
    <property name="xxx" value="xxx" />
    ...
</bean>
```

**property** 元素的 <font color="0088dd">**name**</font> 属性用于指定对象的属性名，<font color="0088dd">**value**</font> 属性用于指定要设置值。

为了简化配置，Spring 提供了一个名为 <font color="0088dd">**p**</font> 的 schema，来简化配置。

```xml
<beans xmlns="..."
       xmlns:xsi="..."
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="...">

    <bean id="tom" class="com.hemiao.bean.Human" p:name="tom" p:age="20" />

    ...
</beans>
```

使用这种简写方式，完全不用出现 **property** 子元素，只需在 **bean** 元素中多增加几个 <font color="0088dd">**p:属性名="属性值"**</font> 这样的属性。


## 四、Spring 装配引用类型属性

对象的属性不一定都是简单类型，还有可能有**引用类型**，即对象间有“has-a”关系。为引用类型的属性赋值不同于基本类型的属性，不是使用 `value`，而是 `ref` 。


### 1. 构造方法装配

```xml
<bean id="jerry" class="com.hemiao.bean.Cat" p:name="jerry" p:age="2"/>

<bean id="tom" class="com.hemiao.bean.Human">
    <constructor-arg index="0" value="tom"/>
    <constructor-arg index="1" value="20"/>
    <constructor-arg index="2" ref="jerry"/>
</bean>
```


构造方法装配的简写形式中，对于引用类型必须写成： <font color="0088dd">**c:_索引-ref="参数值"**</font> 。

```xml
<bean id="jerry" class="com.hemiao.bean.BlackCat" c:_0="jerry" c:_1="2"/>
<bean id="tom" class="com.hemiao.bean.Human" c:_0="tom" c:_1="20" c:_2-ref="jerry"/>
```


### 2. setter 装配

```xml
<bean id="jerry" class="com.hemiao.bean.Cat" p:name="jerry" p:age="2"/>

<bean id="tom" class="com.hemiao.bean.Human">
    <property name="name" value="tom"/>
    <property name="age" value="20"/>
    <property name="cat" ref="jerry"/>
</bean>
```

属性赋值的简写形式中，对于引用类型必须写成：<font color="0088dd">**p:属性名-ref**</font> 。

```xml
<bean id="jerry" class="com.hemiao.bean.Cat" p:name="jerry" p:age="2"/>
<bean id="tom" class="com.hemiao.bean.Human" p:name="tom" p:age="20" p:cat-ref="jerry"/>
```


## 五、Spring 装配集合类型属性

更复杂的属性类型是集合类型（数组、List、Set、Map）属性。



Bean 的属性可能远不止基本类型这么简单，还有可能是基本类型的集合（List、Set 和 Map）。这种情况下，属性的赋值不再是 `property-value` 这种结构，而是 `property-list-value` 三层结构。

```xml
<bean id="..." class="...">

    <property name="数组属性名">
        <array>
            <value>xxx</value>
            ...
        </array>
    </property>

    <property name="List属性名">
        <list>
            <value>xxx</value>
            ...
        </list>
    </property>

    <property name="Set属性名">
        <set>
            <value>xxx</value>
            ...
        </set>
    </property>

    <property name="Map属性名">
        <map>
            <entry key="xxx" value="xx"/>
            ...
        </map>
    </property>

</bean>
```

如果集合是引用类型的集合，那么使用的子元素就从 **value** 改为 **ref**。map 使用的是 **key-ref** 和 **value-ref** 。

```xml
<list>
    <ref bean="myDataSource" />
</list>

<set>
    <ref bean="myDataSource" />
</set>

<map>
    <entry key ="a ref" value-ref="myDataSource"/>
</map>
```

引用的集合还有一种简写形式

```xml
<list>
    <ref class="类的完全限定名" />
</list>

<set>
    <ref class="类的完全限定名" />
</set>

<map>
    <entry key ="a ref" class="类的完全限定名"/>
</map>
```

## 六、自动装配

当一个 Bean 需要访问另一个 Bean 时，你可以显示指定引用装配它。不过，Spring IoC 容器提供自动装配功能，只需要在 **bean** 的 <font color="0088dd">**autowire**</font> 属性中指定自动装配模式就可以了。

|装配模式|说明|
|:-|:-|
|no|默认值。不执行自动装配。你必须显示地装配所依赖对象|
|byName|以 Bean 的属性名为依据，装配一个与属性名同名的 Bean|
|byType|以 Bean 的属性类型为依据，装配一个与之同类型的 Bean|
|constructor|通过构造方法初始化 Bean 的属性，并依据参数的类型，装配一个与参数同类型的 Bean|

尽管自动装配很强大，但是代价是降低了 Bean 配置的可读性。在实践中，建议仅在依赖关系不复杂的应用中使用。


## 七、注解替代 XML 配置

在 XML 配置文件中加上 `<context:component-scan base-package="Bean所在的包路径"/>` 即可开启 Spring 的自动扫描功能，这是使用注解替代XML配置的*前提* 。

### 1. @Component 注解

<font color="0088dd">**@Component**</font> 注解用于标注于 Bean 的类上。凡是被标注了该注解的类（只要在扫描路径下）都会被 Spring 创建。

@Component 注解有唯一的属性 <font color="0088dd">**value**</font> 属性。它用来为 Bean 命名。

@Component 注解有三个语义化的子注解：

  > -  <b><font color="0088dd">@Repository</font></b>（用于持久层）
  > -  <b><font color="0088dd">@Service</font></b> （用于业务层）
  > -  <b><font color="0088dd">@Controller</font></b>（用于 Web 层）

### 2. @Value 注解

<font color="0088dd">**@Value**</font> 注解用于标注于**简单类型**属性上。凡是被标注了该注解的属性都会被 Spring 注入值（赋值）。

@Value 注解有唯一的属性 <font color="0088dd">**value**</font> 属性。它用来为简单属性指定值。


### 3. @Autowired 注解

<font color="0088dd">**@Autowired**</font> 注解用于标注于**引用类型**属性上。凡是被标注了该注解的属性都会被 Spring 以 <font color="0088dd">**类型**</font> 为依据注入另一个 Bean 的引用。

@Autowired 注解有唯一的属性 <font color="0088dd">**required**</font> 属性（默认值为 `true`）。它用来指示该对该属性的注入是否为必须（默认*必须*），即，在 Spring IoC 容器中没有发现符合类型的其他Bean时，会抛出异常。


### 4. @Qualifier 注解

<font color="0088dd">**@Qualifier**</font> 注解需要结合 **@Autowired** 注解使用。它用于标注于引用类型属性上。凡是被标注了该注解的属性都会被 Spring 以 <font color="0088dd">**名字**</font> 为依据注入另一个 Bean 的引用。

@Qualifier 注解有唯一的属性 <font color="0088dd">**value**</font> 属性。它用于指示需要注入的另一个 Bean 的名字。

## 八、Bean 的作用域

默认情况下，Spring IoC 容器只会对一个 Bean 创建一个实例。即单例。Spring IoC 提供了 4 种 <font color="0088dd">**作用域**</font>，它决定了 Spring IoC 是否/何时 生成一个新的对象。常见有：

- singleton（单例）：默认值。在整个应用中，Spring 只为其生成一个 Bean 的实例。
- prototype（原型）：Spring 每次都会生成一个 Bean 的实例。

在 XML 配置文件中， 通过 bean 元素的 <font color="0088dd">**scope**</font> 属性进行设置。该属性取值：`singleton` | `prototype` | 其他 。

在 注解 配置中，使用 <font color="0088dd">**@Scope**</font> 注解，该注解标注于 Bean 的类上（常见于 @Component 之下）。该注解有唯一属性 <font color="0088dd">**value**</font> 属性，其取值有： `singleton` | `prototype` | 其他。

## 九、使用 Spring 表达式（Spring EL）

Spring 表达式可以出现在 @Value 注解中，用以表达有逻辑关系的赋值。Spring 表达式的基本语法是 `#{表达式}`，即 `@Value("#{表达式}")` 。

Spring EL 中，数字常量直接书写，即 `@Value("#{9527}")`。

Spring EL 中，字符串常量使用单引号（'）括起来，即 `@Value("#{'Hello World'}")` 。

Spring EL 中，对象的引用直接书写对象名，即 `@Value('#{tom}')` 。

在 Spring EL 中，使用对象的属性和方法就像书写点语法表达式。
`@Value("#{tom.age}")`
`@Value("#{tom.getName().toUpperCase()}")`


有时候我们可能希望使用一些静态方法和常量。在 Spring EL 中，书写方式如下：
`@Value("#{T(Math).PI}")`
`@Value("#{T(Math).random()}"`

Spring EL 表达是中可以使用运算符进行运算，包括：算术运算、字符串拼接、逻辑运算、比较运算、甚至是可以使用三元运算符。
