## activiti不使用starter整合

1. 引入依赖

   ```xml
     <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-spring</artifactId>
           <version>6.0.0</version>
       </dependency>
   ```

2. 添加配置

   ```properties
   spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
   spring.datasource.url=jdbc:mysql://localhost:3306/activiti
       ?useUnicode=true&
       characterEncoding=utf-8&
       useSSL=false&
       serverTimezone=UTC&
       nullCatalogMeansCurrent=true
   spring.datasource.username=root
   spring.datasource.password=123456
   ```

   需要注意的是，在url中，添加了针对数据库的条件，其中最后一条`nullCatalogMeansCurrent=true`非常重要，至于有什么用就不概述了，但是没有这条语句的话就无法自动创建对应的二十八张表.

3. 添加配置类

   ```java
      @Configuration
   public class ActivitiConfig {
       /*
       * 配置分为以下几步骤
       * 1. 创建ActivitiConfig
       * 2. 使用ActivitiConfig创建ProcessEngineFactoryBean
       * 3. 使用ProcessEngineFactoryBean创建ProcessEngine对象
       * 4. 使用ProcessEngine对象创建需要的服务对象
       * */
       private Logger logger = LoggerFactory.getLogger(ActivitiConfig.class);
   
       private final DataSource dataSource;
   
       private final PlatformTransactionManager platformTransactionManager;
       @Autowired
       public ActivitiConfig(DataSource dataSource, PlatformTransactionManager platformTransactionManager) {
           this.dataSource = dataSource;
           this.platformTransactionManager = platformTransactionManager;
       }
   /*
   * 1. 创建配置文件，也就是提供一些配置信息，这样就可以自定义自己的创建信息了
   * 需要一些参数，1. 数据源。2. 事务管理器。
   * 这里还加入了自动扫描process包下的bpmn(流程定义文件)的设置，这样就可以省去了部署
    * */
       @Bean
       public SpringProcessEngineConfiguration springProcessEngineConfiguration() {
           SpringProcessEngineConfiguration spec = new SpringProcessEngineConfiguration();
           spec.setDataSource(dataSource);
           spec.setTransactionManager(platformTransactionManager);
           spec.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
           Resource[] resources = null;
           // 启动自动部署流程
           try {
               resources = new PathMatchingResourcePatternResolver().getResources("classpath*:process/*.bpmn");
           } catch (IOException e) {
               e.printStackTrace();
           }
           spec.setDeploymentResources(resources);
           return spec;
       }
   
       @Bean
       public ProcessEngineFactoryBean processEngine() {
           ProcessEngineFactoryBean processEngineFactoryBean = new ProcessEngineFactoryBean();
           processEngineFactoryBean.setProcessEngineConfiguration(springProcessEngineConfiguration());
           return processEngineFactoryBean;
       }
   
       @Bean
       public RepositoryService repositoryService() throws Exception {
           return processEngine().getObject().getRepositoryService();
       }
   
       @Bean
       public RuntimeService runtimeService() throws Exception {
           return processEngine().getObject().getRuntimeService();
       }
   
       @Bean
       public TaskService taskService() throws Exception {
           return processEngine().getObject().getTaskService();
       }
   
       @Bean
       public HistoryService historyService() throws Exception {
           return processEngine().getObject().getHistoryService();
       }
   }
   ```



## 使用starter整合

1. 引入依赖

   ```xml
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-spring-boot-starter</artifactId>
           <version>7.1.0.M3.1</version>
       </dependency>
   ```

2. 添加配置

   ````properties
   spring.activiti.database-schema-update=true
   spring.activiti.process-definition-location-prefix=classpath:process/*.bpmn
   # 这里还是记得注意最后面添加url时添加的语句
   spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
   spring.datasource.url=jdbc:mysql://localhost:3306/activiti?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC&nullCatalogMeansCurrent=true
   spring.datasource.username=root
   spring.datasource.password=root
   ````

   这个方案有点缺陷，默认不创建history对应的表,解决方法由于版本过新暂未找到。