# Mavan 父子模块（继承）

继承是 Maven 中很强大的一种功能，继承可以使得子 POM 可以获得 parent 中的各项配置，可以对子 pom 进行统一的配置和依赖管理。父 POM 中的大多数元素都能被子 POM 继承，这些元素包含：

- groupId
- version
- description
- url
- inceptionYear
- organization
- licenses
- developers
- contributors
- mailingLists
- scm
- issueManagement
- ciManagement
- properties
- dependencyManagement
- dependencies
- repositories
- pluginRepositories
- build
- plugin executions with matching ids
- plugin configuration
- etc.
- reporting
- profiles

注意下面的元素，这些都是不能被继承的。

- artifactId
- name
- prerequisites


## 父项目

通常 Maven 的父项目是一个特殊项目，本质上，它是因为为了复用配置而存在的项目，而并非一个实际意义上的真实的项目。

父项目的 `package` 类型是 `pom` 类型，既非 `jar` 类型，又非 `war` 类型。

父项目并不是一个真正的项目，因此父项目下完全不需要存在 `java` 包和 `resources` 包，父项目下只需要存在一个 `pom.xml` 文件即可。


## 子项目

为一个子项目指定父项目只需要在其 pom.xml 中添加 `<parent>` 元素即可。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>my-parent</artifactId>
        <version>2.0</version>
        <relativePath>../my-parent</relativePath>
    </parent>
    <artifactId>my-project</artifactId>
</project>
```

其中 `relativePath` 元素不是必须的，指定后会优先从指定的位置查找父 pom （而后是本地仓库、私服和中央仓库）。


> `NOTE`：子项目不一定非得新建在父项目内。


## 父子项目的使用

通常，让子项目重用父项目的配置，有两种表现形式：

1. 父项目中定义 `<dependencies>`，子项目『天然』继承了父项目的 `<dependences>`，从而不再需要引入这些依赖。
2. 父项目通过配置 `<dependencyManagement>` 定义依赖包的版本，子项目不用则已，一旦使用这些包中的某个/某些，则不再需要指定它们的版本信息。


