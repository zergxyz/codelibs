# Spring (Boot) Configurations

some notes for spring framework configuration. 

**Error configuring: org.springframework.boot:spring-boot-maven-plugin. Reason: Unable to retrieve component configurator for plugin configuration
AN: you need to upgrade the maven to version 3.x to make the spring-boot maven plug-in work correctly.**

Spring logging
You can use bom ( bill of materials ) and add it to your dependent management section to make sure all your spring dependencies including transitive ones are using the same  version.
Once you add the bom, you no longer need to specify the version of spring
Spring uses JCL for logging. In maven it's the common-logging module.
How to switch the comon-login? 

```xml
    <dependencies>
    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.2.4.RELEASE</version>
    <exclusions>
    <exclusion>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    </exclusion>
    </exclusions>
    </dependency>
    </dependencies>
```


DI :

	* Constructor based
	* Setter based

In spring application, we have 2 ways of IOC, usually DI is preferred if it’s possible (for example Spring MVC will do this for you) but sometimes we have to use dependency lookup to programmatically get the dependency if you use spring as a standalone java program through bean factory.

application context is an extension of bean factory, not only provide DI but other services like AOP, messages service and event handling.
An example of using application context to register and get bean can be found in this note: Example of @EnableScheduling in Spring

@autowired == @inject == Resource (name = beanname).
@autowired and @value(sas) can be only applied to one constructor for DI if the class has more than one constructors.

Basic boot configuration
you just need to have spring boot starter in your maven pom as the parent dependency. Then add whatever you need it like[1]:

	* spring boot starter web for the rest and mvc
	* spring boot starter batch for the spring batch
	* starter-ampq for the rabbitMQ

In the main method, Class SpringApplication is a boot specific component and the default entry for most of the spring boot applications. It will take care of spring application context for us [1].
For maven package, it will generate 2 jars and The .original file is the output of the default Maven compiler, while the other one is the jar file enhanced by Spring Boot’s plugin.

About the property files, you can use application.properties/yaml to override the default app settings. More information can be found in the spring doc [2]. Default configs can be found in the org.springframework.boot.autoconfigure.* packages

Use Java configuration to define a JNDI datasource: 
``` java 
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories
@PropertySource("classpath:props.properties")
public class DataSourceConfigProd {

    @Autowired
    private Environment env;

    @Bean
    public AbstractPlatformTransactionManager transactionManager() {
        return new JpaTransactionManager(entityManagerFactory());
    }

    @Bean
    public EntityManagerFactory entityManagerFactory() {
        final LocalContainerEntityManagerFactoryBean bean = new LocalContainerEntityManagerFactoryBean();
        bean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        bean.setDataSource(dataSource());
        final Properties props = new Properties();
        props.setProperty("hibernate.dialect", env.getProperty("hibernate.dialect"));
        props.setProperty("hibernate.hbm2ddl.auto", env.getProperty("hibernate.hbm2ddl.auto"));
        props.setProperty("hibernate.show_sql", env.getProperty("hibernate.show_sql"));
        props.setProperty("hibernate.format_sql", env.getProperty("hibernate.format_sql"));

        bean.setJpaProperties(props);
        return bean.getObject();
    }

    @Bean
    public DataSource dataSource() {
        final JndiDataSourceLookup dsLookup = new JndiDataSourceLookup();
        dsLookup.setResourceRef(true);
        DataSource dataSource = dsLookup.getDataSource("jdbc/yourJdbcGoesHere");
        return dataSource;
    }
}
```