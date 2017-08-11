### spring boot 整合Hibernate

Spring boot整合Hibernate时Dao层是一个接口继承JpaRepository，JpaRepository中已经包含一些已经可以使用的增删改查的方法。详细内容见Spring data jpa 查询介绍

http://www.ityouknow.com/springboot/2016/08/20/springboot%28%E4%BA%94%29-spring-data-jpa%E7%9A%84%E4%BD%BF%E7%94%A8.html

​	Spring data jpa 动态查询详见

http://www.importnew.com/24514.html

#### 两种实现

##### 第一种（自动配置）

​	属性文件要满足spring boot 给出的配置文件的规范

```
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8  
spring.datasource.username=root  
spring.datasource.password=941005  
spring.datasource.driverClassName=com.mysql.jdbc.Driver  
spring.jpa.database = MYSQL  
spring.jpa.hibernate.ddl-auto=update  
spring.jpa.show-sql = true  
spring.jpa.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect  
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
```

​	减轻Dao层开发，JpaRepository中有实现了多个简单的增删改查

```java
//Spring会为UserDao生成一个代理对象
public interface UserDao extends JpaRepository<Person, Integer>{
	Person findByName(String name);
}
```

​	Service层开发

```java
@Service("userService")
public class UserServiceImpl implements UserService{
	//这里注入的是一个代理对象
  	@Autowired
	private UserDao userDao;
	@Override
	public Person get(String id) {
		// TODO Auto-generated method stub
		System.out.println(userDao);
		return userDao.findOne(Integer.parseInt(id));
	}
	@Override
	public boolean save(Person user) {
		// TODO Auto-generated method stub
		userDao.save(user);
		return false;
	}

}
```

##### 第二种（使用配置类实现）

配置类如下

```java
@Configuration
@EnableTransactionManagement
public class HibernateConfig {
	@Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test?characterEncoding=utf8");
        dataSource.setUsername("root");
        dataSource.setPassword("941005");

        return dataSource;
    }
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactoryBean.setDataSource(dataSource());
        //扫描实体包
        entityManagerFactoryBean.setPackagesToScan("com.lzj.domain");
        entityManagerFactoryBean.setJpaProperties(buildHibernateProperties());
        entityManagerFactoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter() {{
            setDatabase(org.springframework.orm.jpa.vendor.Database.MYSQL);
        }});
        return entityManagerFactoryBean;
    }

    protected Properties buildHibernateProperties()
    {
        Properties hibernateProperties = new Properties();

        hibernateProperties.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL5Dialect");
        hibernateProperties.setProperty("hibernate.show_sql", "true");
        hibernateProperties.setProperty("hibernate.use_sql_comments", "false");
        hibernateProperties.setProperty("hibernate.format_sql", "true");
        hibernateProperties.setProperty("hibernate.hbm2ddl.auto", "update");
        hibernateProperties.setProperty("hibernate.generate_statistics", "false");
        hibernateProperties.setProperty("javax.persistence.validation.mode", "none");

        //Audit History flags
        hibernateProperties.setProperty("org.hibernate.envers.store_data_at_delete", "true");
        hibernateProperties.setProperty("org.hibernate.envers.global_with_modified_flag", "true");

        return hibernateProperties;
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new JpaTransactionManager();
    }

    @Bean
    public TransactionTemplate transactionTemplate() {
        return new TransactionTemplate(transactionManager());
    }
}
```