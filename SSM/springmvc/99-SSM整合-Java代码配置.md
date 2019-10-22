
SpringMVC 会去找项目中所有的 DispatcherServletInitialize 的实现类，这些类中的代码就等价于在 web.xml 中的某些配置信息。

```java
/**
 * 该类的存在相当于配置了 web.xml 中 DispatcherServlet 和 配置文件加载
 */
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	private final static Logger LOG = LoggerFactory.getLogger(WebAppInitializer.class);

	@Override
	protected Class<?>[] getRootConfigClasses() {
		LOG.info("------root配置类初始化------");
		return new Class<?>[] { SpringServiceConfig.class, SpringDaoConfig.class, SpringSecurityConfig.class };
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		LOG.info("------web配置类初始化------");
		return new Class<?>[] { SpringWebConfig.class };
	}

	@Override
	protected String[] getServletMappings() {
		LOG.info("------映射根路径初始化------");
		return new String[] { "/" };// 请求路径映射，将路径映射到DispatcherServlet上
	}

  @Override
	protected Filter[] getServletFilters() {
		return new Filter[] {
				new CharacterEncodingFilter("UTF-8", true)
		};
	}
}
```

SpringDaoConfig.java 等价于 spring-dao.xml

```java
@Configuration
@MapperScan("com.xja.hemiao.dao")
public class SpringDaoConfig {

	@Bean(name = "dataSource")
	public DataSource getDataSource() {
		Properties properties = new Properties();
		properties.setProperty("driverClassName", "com.mysql.jdbc.Driver");
		properties.setProperty("jdbcUrl", "jdbc:mysql://localhost:3306/scott");
		properties.setProperty("username", "root");
		properties.setProperty("password", "123456");

		HikariConfig config = new HikariConfig(properties);

		return new HikariDataSource(config);
	}

	@Bean(name = "sqlSessionFactory")
	public SqlSessionFactoryBean getSqlSessionFactory() throws IOException {

		PathMatchingResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();

		SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
		factory.setDataSource(getDataSource());
		factory.setConfigLocation(patternResolver.getResource("classpath:mybatis/mybatis-config.xml"));
		factory.setMapperLocations(patternResolver.getResources("classpath:mybatis/mapper/*Mapper.xml"));
		factory.setTypeAliasesPackage("com.xja.hemiao.bean");

		return factory;
	}
}
```


SpringServiceConfig.java 等价于 spring-service.xml

```java
@Configuration
@ComponentScan(basePackages = "com.xja.hemiao.service")
@EnableTransactionManagement
public class SpringServiceConfig {

	@Bean(name = "transactionManager")
	public DataSourceTransactionManager transactionManager(DataSource dataSource) {
		return new DataSourceTransactionManager(dataSource);
	}
}
```

SpringWebConfig.java 等价于 spring-web.xml

```java
@Configuration
@EnableWebMvc
@ComponentScan("com.xja.hemiao.web.controller") // 包扫描
public class SpringWebConfig implements WebMvcConfigurer {

	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/jsp");
		resolver.setSuffix(".jsp");
		return resolver;
	}

	/*
	@Bean(name = "multipartResolver") // bean必须写name属性且必须为multipartResolver
	protected CommonsMultipartResolver multipartResolver() {
		CommonsMultipartResolver commonsMultipartResolver = new CommonsMultipartResolver();
		commonsMultipartResolver.setMaxUploadSize(5 * 1024 * 1024);
		commonsMultipartResolver.setMaxInMemorySize(0);
		commonsMultipartResolver.setDefaultEncoding("UTF-8");
		return commonsMultipartResolver;
	}
	*/

	// 静态资源的处理
	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}

	 /*  @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/css/**").addResourceLocations("classpath:/css");
        registry.addResourceHandler("/js/**").addResourceLocations("classpath:/js");
        registry.addResourceHandler("/img/**").addResourceLocations("classpath:/img");
    }*/


	/*
	@Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        super.configureMessageConverters(converters);

        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();

        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(
                SerializerFeature.PrettyFormat
        );
        fastConverter.setFastJsonConfig(fastJsonConfig);

        converters.add(fastConverter);
    }
	 */
}
```
