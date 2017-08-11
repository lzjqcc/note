## Spring boot 整合mybatis

Spring boot 整合mybatis没有spring的XML配置文件。通过类的方式将mybatis的SqlSessionFactory，DataSources加入到Spring工厂中

Spring boot 会将application.properties文件**属性封装成**org.springframework.core.env.Environment 类，

在mybatis的接口文件中加上@Mapper注解



#### 第一步（数据源）

在application.properties中添加数据源配置信息

```properties
cluster.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
cluster.datasource.username=root
cluster.datasource.password=941005
cluster.datasource.driverClassName=com.mysql.jdbc.Driver
```

#### 第二步（配置类）

创建配置类

@Primary 标志这个 Bean 如果在多个同类 Bean 候选时，该 Bean 优先被考虑。「多数据源配置的时候注意，必须要有一个主数据源，用 @Primary 标志该 Bean」

@MapperScan 扫描 Mapper 接口并容器管理，包路径精确到 master，为了和下面 cluster 数据源做到精确区分

@Value 获取全局配置文件 application.properties 的 k-v 配置,并自动装配

**@Qualifier根据name进行注入，@Autowired 根据类型进行注入（这两个可以配合使用）**  

sqlSessionFactoryRef 表示定义了 key ，表示一个唯一 SqlSessionFactory 实例

```java
package org.spring.springboot.config.ds;

import com.alibaba.druid.pool.DruidDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;
//相当于Spring中XML配置文件
@Configuration
// 扫描 Mapper 接口并容器管理
@MapperScan(basePackages = MasterDataSourceConfig.PACKAGE, sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MasterDataSourceConfig {

    // 精确到 master 目录，以便跟其他数据源隔离
    static final String PACKAGE = "org.spring.springboot.dao.master";
    static final String MAPPER_LOCATION = "classpath:mapper/master/*.xml";    

    @Value("${master.datasource.url}")
    private String url;

    @Value("${master.datasource.username}")
    private String user;

    @Value("${master.datasource.password}")
    private String password;

    @Value("${master.datasource.driverClassName}")
    private String driverClass;

    @Bean(name = "masterDataSource")
    @Primary
    public DataSource masterDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClass);
        dataSource.setUrl(url);
        dataSource.setUsername(user);
        dataSource.setPassword(password);
        return dataSource;
    }
    
    @Bean(name = "masterTransactionManager")
    @Primary
    public DataSourceTransactionManager masterTransactionManager() {
        return new DataSourceTransactionManager(masterDataSource());
    }

    @Bean(name = "masterSqlSessionFactory")
    @Primary
    public SqlSessionFactory masterSqlSessionFactory(@Qualifier("masterDataSource") DataSource masterDataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(masterDataSource); 
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver() 
                .getResources(MasterDataSourceConfig.MAPPER_LOCATION)); 
        return sessionFactory.getObject(); 
    }
}
```

#### 第三步（mybatis接口,XML）

mybatis的接口

```java
//@Mapper 表示mybatis会为该接口生成一个实现类
@Mapper
public interface UserDao {

    /**
     * 根据用户名获取用户信息
     *
     * @param userName
     * @return
     */
    User findByName();
}
```

mybatis XML文件

```xml
<mapper namespace="org.spring.springboot.dao.master.UserDao">
	<resultMap id="BaseResultMap" type="org.spring.springboot.domain.User">
		<result column="id" property="id" />
		<result column="name" property="userName" />
		<result column="address" property="description" />
	</resultMap>
	<parameterMap id="User" type="org.spring.springboot.domain.User"/>
	<select id="findByName" resultMap="BaseResultMap" >
		select * from user where id=1;
	</select>
</mapper>

```

#### 最后

Spring boot 测试

首先在pom文件中添加

```xml
<dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
</dependency>
```

```java
//指定SpringBoot启动类
@SpringBootTest(classes=Application.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class TestConnection {
	@Autowired
	private UserService userService;
	@Test
	public  void first() throws Exception {
		// TODO Auto-generated method stub
		Class.forName("com.mysql.jdbc.Driver");
		Connection connection=DriverManager.getConnection("jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8", "root", "941005");
		PreparedStatement statement=connection.prepareStatement("select * from user");
		ResultSet set=statement.executeQuery();
		if (set.next()) {
			System.out.println("success");
		}
	}
	@Test
	public void test(){
		System.out.println(userService.getAllUsers());
	}
}
```