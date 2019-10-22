# MyBatis Generator 和 Example 对象

## Mybatis Generator

### 1. 图形化界面操作

Mybatis 属于 **半自动 ORM 框架**，在使用这个框架中，工作量最大的就是书写 Mapping 的映射文件，由于手动书写很容易出错，为此 Mybatis 官方提供了个 Mybatis-Generator 来帮我们自动生成文件。

早期的 Mybatis Generator 的使用十分原始，使用起来非常麻烦，在开源社区的努力下，截止目前为止，已经出现了图形化界面的使用方式，已经变得十分便捷（屏蔽掉了大量繁琐的生涩的配置操作）。

现在使用较多的图形化工具是 [mybatis-generator-gui](https://github.com/zouzg/mybatis-generator-gui)

使用方式十分简单：

```bash
git clone https://github.com/zouzg/mybatis-generator-gui
cd mybatis-generator-gui
mvn jfx:jar
cd target/jfx/app/
java -jar mybatis-generator-gui.jar
```

![mybatis-generator-gui-1](../img/mybatis-generator-gui-1.png)

![mybatis-generator-gui-2](../img/mybatis-generator-gui-2.png)

![mybatis-generator-gui-3](../img/mybatis-generator-gui-3.png)


本工具由于使用了 Java 8 的众多特性，所以要求 JDK 1.8.0.60 以上版本，另外 JDK 1.9 暂时还不支持。


### 2. mybatis-generator 原始的使用方式（了解）

Mybatis-generator 的使用流程非常简单：

- 编写配置文件
- 编写代码去读取配置文件，并通过特定方法以此配置文件为线索，生成代码和配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>

  <!--数据库驱动-->
  <classPathEntry location="mysql-connector-java-5.1.26.jar" />

  <context id="test" defaultModelType="flat">

    <commentGenerator>
      <property name="suppressDate" value="true"/>        <!-- 是否去除生成的注释中的日期部分。true 表示去掉-->
      <property name="suppressAllComments" value="true"/> <!-- 是否去除自动生成的注释。 true 表示去掉 -->
    </commentGenerator>

    <!-- 数据库链接四大配置 -->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver" 
        connectionURL="jdbc:mysql://localhost/scott" 
        userId="root" 
        password="123456"/>

    <javaModelGenerator targetProject="src" targetPackage="bean.po" />      <!-- 指定生成 po 的位置，及其包名 -->
    <sqlMapGenerator targetProject="src" targetPackage="mybatis.mapper" />  <!-- 指定生成 mapper 文件的位置 -->
    <javaClientGenerator type="XMLMAPPER" targetProject="src" targetPackage="dao" /> <!-- 指定生成 dao 的位置，及其包名 -->

    <!-- 要生成哪些表 -->
    <table tableName="dept" domainObjectName="Department"
        enableCountByExample="false" enableUpdateByExample="false"
        enableDeleteByExample="false" enableSelectByExample="false"/>

    <table tableName="t_user" domainObjectName="User"
        enableCountByExample="false" enableUpdateByExample="false"
        enableDeleteByExample="false" enableSelectByExample="false">
      <property name="useActualColumnNames" value="false"/>       <!-- 取消直接使用列名，即使用驼峰命名 -->
      <columnRenamingRule searchString="^t_" replaceString=""/>   <!-- 去掉列名固定前缀 -->
    </table>

    <table ... />
    <table ... />
    <table ... />

  </context>

</generatorConfiguration>
```

### 3. 生成代码和映射文件有两种方式：

本质上其实是一样的。

方式一：

在任意文件家下准备如下内容：

- generatorConfig.xml 文件
- mybatis-generator-core-1.3.5.jar 包
- mysql-connector-java-5.1.26.jar 包
- src 空目录

> 注意：<small>使用较低版本的 mysql 驱动。从 mysql 驱动 v6 开始，有些设置发生了变化，会导致有些配置变得更严谨（也更繁琐）。</small>

在当前目录下执行

```shell
$ java -jar ./mybatis-generator-core-1.3.5.jar -configfile ./generatorConfig.xml -overwrite
```

方式二：

另建一个 java 项目，引入 mybatis-generator 和 mysql-connector-java 包，编写并执行如下代码。

```java
String rootPath = this.getClass().getResource("/generatorConfig.xml").getFile().toString();
File configFile = new File(rootPath);

List<String> warnings = new ArrayList<String>();
boolean overwrite = true;
ConfigurationParser cp = new ConfigurationParser(warnings);
Configuration config = cp.parseConfiguration(configFile);
DefaultShellCallback callback = new DefaultShellCallback(overwrite);
MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
myBatisGenerator.generate(null);
```

## Mybatis Example 对象

```java
EmployeeExample example = new EmployeeExample();

example.createCriteria().andEmpnoEqualTo(7369); // 情况一

example.or(example.createCriteria().andEmpnoEqualTo(7499)); // 情况二

example.or(example.createCriterial().xxxx());   // 情况三

// 一个条件一个 Criterial 对象。从第二个条件开始，调用 example.or() 

List<Employee> list = dao.selectByExample(example);
System.out.println(list.size());
```

```java
TestTableExample example = new TestTableExample();

example.or()
  .andField1EqualTo(5)
  .andField2IsNull();

example.or()
  .andField3NotEqualTo(9)
  .andField4IsNotNull();

List<Integer> field5Values = new ArrayList<Integer>();
field5Values.add(8);
field5Values.add(11);
field5Values.add(14);
field5Values.add(22);

example.or()
  .andField5In(field5Values);

example.or()
  .andField6Between(3, 7);
```

相当于

```sql
where (field1 = 5 and field2 is null)
     or (field3 != 9 and field4 is not null)
     or (field5 in (8, 11, 14, 22))
     or (field6 between 3 and 7);
```