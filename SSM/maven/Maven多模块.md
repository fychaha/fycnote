# Maven 多模块（聚合）

如果说，Maven 继承（父子项目）的目的为了配置文件的复用和配置信息的统一管理，那么 Maven 多模块（聚合）项目的目的是项目功能上的拆分。

例如， 在 log4j1 时代，log4j 项目的『产出物』只有一个 `log4j.jar` 包。到了 log4j2 时代，log4j 项目的『产出物』就变成了两个 `log4j-api` 和 `log4j-core`。很明显，就是两部分相对独立的代码分别打成了两个包，而并不像以前那样打成一个包。

Maven 聚合项目和父子项目一样，也是一个『假』项目包含一个或多个真项目，不过有几点不同：

1. 虽然也是要使用 `<parent>`，但是聚合项目中的“父项目”还会多出来一个 `<modules>` 元素。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>my-parent</artifactId>
    <version>2.0</version>
    <packaging>pom</packaging>
    <modules>
        <module>my-project</module>
        <module>another-project</module>
        <module>...</module>
        <module>...</module>
        <module>...</module>
        </modules>
    ...
```

在列出模块时，不需要自己考虑模块间依赖关系，即 POM 给出的模块排序并不重要。Maven 『捋清楚』它们之间的依赖关系。

2. 通常多模块项目，module 是在 project 里面的（父子项目通常不要求这个）。






