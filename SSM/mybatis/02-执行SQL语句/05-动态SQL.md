# 动态 SQL

简而言之，动态 SQL 就是在 Mapper 中使用分支、循环等逻辑。常见的动态 SQL 元素包括：

- if 元素
- choose-when-otherwise 元素
- where 元素
- set 元素
- foreach 元素

## 一、if 元素

if 元素是我们最常见的元素判断语句，相当于 Java 中的 if 语句。它的 test 属性是它的必要属性。

```xml
<select id="select" parameterType="int" resultType="Department">
  SELECT deptno, dname, loc FROM dept
  <if test="deptno != null">
    WHERE deptno = #{deptno}
  </if>
</select>
```

## 二、choose-when-otherwise 元素

MyBatis 并未提供类似 if-else 元素来处理分支情况，if 元素可出现多次，但它们是并列的判断，而非互斥的判断。

choose-when-otherwise 元素类似于 Java 中的 switch-case，用于处理多个条件间的互斥判断。

```xml
<select id="selectBySallary" resultType="Employee">
  SELECT empno, ename, job, mgr, hiredate, sal, comm, deptno FROM emp
  <choose>
    <when test="min != null and max != null">
      WHERE sal >= #{min} AND sal <= #{max}
    </when>
    <when test="min != null">
      WHERE sal >= #{min} 
    </when>
    <when test="max != null">
      WHERE sal <= #{max}
    </when>
    <otherwise></otherwise>
  </choose>
</select>
```

## 三、where 元素

如果我们强行规定，上述 choose-when-otherwise 所实现的功能必须使用 if 实现，那么将会写成如下形式：

```xml
<select id="selectBySallary" resultType="Employee">
  SELECT empno, ename, job, mgr, hiredate, sal, comm, deptno FROM emp
  WHERE 1 = 1
  <if test="min != null">
    AND sal >= #{min}
  </if>
  <if test="max != null">
    AND sal <= #{max}
  </if>
</select>
```

注意体会上面 `WHERE 1 = 1` 的位置及其作用。

由于判断条件有可能有，也可能没有，所有在 if 元素中，WHERE 关键字出现的地方就有些“尴尬”。`WHERE 1 = 1` 就是此问题的 ***非典型*** 解决方案。

MyBatis 提供了 Where 元素以解决上述尴尬问题。

```xml
<select id="selectBySallary" resultType="Employee">
  SELECT empno, ename, job, mgr, hiredate, sal, comm, deptno FROM emp
  <where>
    <if test="min != null">
      AND sal >= #{min}
    </if>
    <if test="max != null">
      AND sal <= #{max}
    </if>
  </where>
</select>
```

## 四、set 元素

类似于 where 的元素，set 元素对应于 SQL 语句中的 SET 子句。它专用于 update 语句，用于包含所需更新的列。

set 元素常常和 if 元素联合使用。因为在“选择性更新”功能中，有一个`最后一个逗号`问题。

`注意`，<small>更新行为务必要保证更新至少一个属性，否则 MyBatis 更新语句提示 update 语句错误</small>

```xml
<update id="updateByPrimaryKeySelective" parameterType="Department">
  UPDATE dept
  <set>
    <if test="dname != null"> dname = #{dname}, </if>
    <if test="loc != null"> loc = #{loc}, </if>
  </set>
  WHERE deptno = #{deptno}
</update>
```

## 五、foreach 元素

foreach 元素使用不多，但是当需要构建包含 IN 子句的查询时，则必用到。

```xml
<select id="selectInDeptnos" resultType="Employee">
  SELECT empno, ename, job, mgr, hiredate, sal, comm, deptno FROM emp 
  WHERE deptno IN 
  <foreach collection="list" item="deptno" open="(" separator="," close=")">
    #{deptno}
  </foreach>
</select>
```

collection 属性表示集合类型，其属性值可以是 list 或 array，对应参数类型为 List 或 数组。
