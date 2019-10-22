# 基本概念

Mapper 是 MyBatis 最强大的工具与功能，它用于执行SQL语句。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="...">
  <insert id="..."> ... </insert>
  <delete id="..."> ... </delete>
  <update id="..."> ... </update>
  <select id="..."> ... </select>
</mapper>
```

| 元素      | 描述                             | 备注                            |
|:--------- |:------------------------------- |:------------------------------- |
| select    | 查询语句，常用又复杂              | 可以自定义参数，返回结果集等      |
| insert    | 插入语句                         | 执行后返回一个整数，代表插入的条数 |
| update    | 更新语句                         | 执行后返回一个整数，代表更新的条数 |
| delete    | 删除语句                         | 执行后返回一个整数，代表删除的条数 |
| resultMap | 定义查询结果映射关系，常用又复杂   | 它将提供映射规则                  |

