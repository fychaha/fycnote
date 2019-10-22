# Spring 中各种数据库连接池配置

使用 mysql-connector 6+ 连接 MySQL 时『多』出来两个小要求：

- MySQL 驱动类发生了变化。原来的 Driver 被标注为废弃，改用新的 Driver：`com.mysql.cj.jdbc.Driver`
- 需要指定时区 `serverTimezone` 参数：`serverTimezone=UTC` 

例如：

```
url=jdbc:mysql://localhost:3306/scott?serverTimezone=UTC&?useUnicode=true&characterEncoding=utf8&useSSL=false
```

```xml
<!-- HikariCP -->
<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
  <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/scott?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=UTF8&amp;useSSL=false" />
  <property name="username" value="root" />
  <property name="password" value="123456" />
</bean>

<!-- Druid -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/scott?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=UTF8&amp;useSSL=false" />
  <property name="username" value="root"/>
  <property name="password" value="123456"/>
</bean>

<!-- DBCP2 -->
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
  <property name="url" value="jdbc:mysql://localhost:3306/scott?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=UTF8&amp;useSSL=false"/>
  <property name="username" value="root" />
  <property name="password" value="123456" />
</bean>
```

