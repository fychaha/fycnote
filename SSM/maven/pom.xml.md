# pom.xml

```xml
<properties>
    <!-- for maven -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>

    <!-- for Base -->
    <lombok.version>1.16.20</lombok.version>
    <log4j2.version>2.9.0</log4j2.version>         <!-- log4j2 日志包 -->
    <spring.version>4.3.2.RELEASE</spring.version>  <!-- Spring 相关 -->
    <aspectj.version>1.8.10</aspectj.version>       <!-- Spring AOP 依赖 -->

    <!-- for Web -->
    <servlet.version>3.1.0</servlet.version>        <!-- Servlet -->
    <jsp.version>2.2</jsp.version>                  <!-- jsp -->
    <jstl.version>1.2</jstl.version>                <!-- jstl -->
    <jackson.version>2.9.9</jackson.version>        <!-- json 解析包 -->
    <hibernate-validator.version>5.4.1.Final</hibernate-validator.version>  <!-- hibernate 实现 jsr303 -->

    <!-- for Dao -->
    <mysql.version>5.1.41</mysql.version>           <!-- mysql 数据库驱动 -->
    <druid.version>1.1.10</druid.version>           <!-- druid 数据库连接池 -->
    <mybatis.version>3.4.1</mybatis.version>        <!-- mybatis -->
    <pagehelper.version>5.1.4</pagehelper.version>  <!-- mybatis 分页组件 -->
    <mybatis-spring.version>1.3.0</mybatis-spring.version> <!-- 用于 spring 整合 Mybatis -->

    <!-- Test -->
    <junit.version>4.12</junit.version>             <!-- junit 测试框架 -->
</properties>

<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
    </dependency>

    <!-- log4j-api、log4j-core 和 slf4j-api 被依赖，自动导入，不再手动指定 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>${log4j2.version}</version>
    </dependency>

    <!-- Spring 的核心包： core、beans、expression、aop 和 context 被依赖，自动导入 -->

    <!-- spring-web 被 spring-webmvc 依赖，自动导入，不再手动指定。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>      <!-- Spring MVC -->
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>        <!-- Spring 整合 Dao 需要 -->
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>          <!-- Spring 事务功能 -->
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjrt</artifactId>          <!-- spring aop 的 aspectj 功能 -->
        <version>${aspectj.version}</version>
    </dependency>

    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>      <!-- spring aop 的 aspectj 功能 -->
        <version>${aspectj.version}</version>
    </dependency>

    <!-- for Java SE -->


    <!-- jackson-annotations 和 jackson-core 被依赖，自动导入，不再手动指定 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>   <!-- jackson 处理 json 辅助部件 -->
        <version>${jackson.version}</version>
    </dependency>

    <!-- for Web -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>  <!-- java web servlet 接口声明 -->
        <version>${servlet.version}</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>            <!-- java web jsp 接口声明 -->
        <version>${jsp.version}</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>               <!-- jsp 上使用 jstl 标签库所 -->
        <version>${jstl.version}</version>
    </dependency>

    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId><!-- jsr303 和 349 的实现-->
        <version>${hibernate-validator.version}</version>
    </dependency>

    <!-- for Dao -->

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>              <!-- druid 数据库连接池 -->
        <version>${druid.version}</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>   <!-- mysql 数据库驱动包 -->
        <version>${mysql.version}</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>     <!-- spring 整合 mybatis -->
        <version>${mybatis-spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>            <!-- mybatis -->
        <version>${mybatis.version}</version>
    </dependency>

    <dependency>                                    <!-- mybatis 分页插件 -->
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>${pagehelper.version}</version>
    </dependency>

    <!-- for Test -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>        <!-- sprint test -->
        <version>${spring.version}</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>              <!-- junit -->
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>

</dependencies>

<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>    <!-- tomcat 7 插件 -->
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <path>/${project.artifactId}</path>
                <uriEncoding>UTF-8</uriEncoding>
            </configuration>
        </plugin>
    </plugins>

    <resources>  <!-- 设定包含 resources 和 java 目录下的资源文件 -->
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```