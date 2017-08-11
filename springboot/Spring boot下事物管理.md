## Spring boot下事物管理

在配置类(MybatisConfig)上使用注解@EnableTransactionManagement开启事物支持

```xml
类似<tx:annotation-driven  transaction-manager="transactionManager">
```

```java
//类似指明spring的配置文件
@Configuration 
//开启事物支持<tx:annotation-driven  transaction-manager="transactionManager">
@EnableTransactionManagement
// 扫描 Mapper 接口并容器管理
@MapperScan(basePackages = "com.lzj.dao", sqlSessionFactoryRef = "sqlSessionFactory")
public class MybatisConfig {
```

在配置类中选择事物类型

```java
//MybatisConfig
@Bean(name="transactionManager")
	public DataSourceTransactionManager getTransactionManager(@Qualifier("dataSource") DataSource dataSource){
		return new DataSourceTransactionManager(dataSource);
	}
```

在业务方法上使用@Transaction配置事物



```java
//UserServiceImpl
@Transactional
	@Override
	public void insert(User user) {
		userDao.insert(user);
		//int i=1/0;
	}
```
注意：

如果Spring容器中存在多个 PlatformTransactionManager 实例，并且没有实现接口 
TransactionManagementConfigurer 指定默认值，在我们在方法上使用注解 @Transactional 
的时候，就必须要用value指定，如果不指定，则会抛出异常。