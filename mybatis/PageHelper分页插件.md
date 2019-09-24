##  集成mybatis分页插件  

1. pom.xml 中引入分页插件依赖   

```
<!-- 分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.5</version>
</dependency>
```

2. 配置文件application.yml 中添加分页配置

   ```
   pagehelper:
     helper-dialect: mysql
     reasonable: true
     support-methods-arguments: true
     params: count==countSql
     page-size-zero: true
   ```

   3. 在service中添加分页查询

      ```java
      public PageInfo<Student> search(int pageNum, int pageSize) {
          PageHelper.startPage(pageNum,pageSize);
          List<Student> list=stduentDao.findAll();
          PageInfo<Student> pageInfo=new PageInfo<>(list);
      
          return pageInfo;
      ```



