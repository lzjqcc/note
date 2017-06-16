## Mybatis OneToOne

​	一个用户对应一个博客 1-1

```
org.apache.ibatis.executor.BaseExecutor //执行增删该查的类
org.apache.ibatis.executor.CachingExecutor
org.apache.ibatis.reflection.MetaObject//存放查询结果的类
org.apache.ibatis.session.Configuration 中resultMaps存放从xml文件中解析出来的column property等元素
```

​	用户表：

```sql
mysql> desc user;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(45) | NO   |     | NULL    |                |
| passward | varchar(45) | NO   |     | NULL    |                |
| email    | varchar(45) | YES  |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+

```

​	博客表：

```sql
mysql> desc blog;
+---------+-------------+------+-----+---------+----------------+
| Field   | Type        | Null | Key | Default | Extra          |
+---------+-------------+------+-----+---------+----------------+
| id      | int(11)     | NO   | PRI | NULL    | auto_increment |
| title   | varchar(45) | YES  |     | NULL    |                |
| content | varchar(45) | YES  |     | NULL    |                |
| user_id | int(11)     | NO   | MUL | NULL    |                |
+---------+-------------+------+-----+---------+----------------+

```

​	用户类：

```java
private Integer id;
private String name;
private String passward;
private String emil;
private Blog blog;
```

​	博客类：

```java
	private Integer id;
	private String title;
	private String content;
```

[^本次查询目的：查询用户信息，并且用户类属性blog含有信息。结果如下所示]: 

```shell
User [id=1, name=李志坚, passward=1234, emil=null, blog=Blog [id=1, title=南昌高校, content=的地方地方]]
```

### mybatis配置文件

#### 	sqlConfig.xml 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 别名定义 -->
    <typeAliases>
        <typeAlias type="com.lzj.study.domain.User" alias="user"/>
        <typeAlias type="com.lzj.study.domain.Blog" alias="blog"/>
        <typeAlias type="com.lzj.study.dao.UserMapper" alias="userMapper"/>
        <typeAlias type="com.lzj.study.dao.BlogMapper" alias="blogMapper"/>
    </typeAliases>

    <!--配置environment环境-->
    <environments default="development">
        <!-- 环境配置1，每个SqlSessionFactory对应一个环境 -->
        <environment id="development1">
            <!-- 事务配置 type= JDBC、MANAGED 1.JDBC:这个配置直接简单使用了JDBC的提交和回滚设置。它依赖于从数据源得到的连接来管理事务范围。   
                2.MANAGED:这个配置几乎没做什么。它从来不提交或回滚一个连接。而它会让容器来管理事务的整个生命周期（比如Spring或JEE应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望这样，因此如果你需要从连接中停止它，将closeConnection属性设置为false -->
            <transactionManager type="JDBC"/>
            <!-- <transactionManager type="MANAGED">
                 <property name="closeConnection" value="false"/>
                 </transactionManager> -->
            <!-- 数据源类型：type = UNPOOLED、POOLED、JNDI 1.UNPOOLED：这个数据源的实现是每次被请求时简单打开和关闭连接。它有一点慢，这是对简单应用程序的一个很好的选择，因为它不需要及时的可用连接。   
                不同的数据库对这个的表现也是不一样的，所以对某些数据库来说配置数据源并不重要，这个配置也是闲置的 2.POOLED：这是JDBC连接对象的数据源连接池的实现，用来避免创建新的连接实例时必要的初始连接和认证时间。   
                这是一种当前Web应用程序用来快速响应请求很流行的方法。 3.JNDI：这个数据源的实现是为了使用如Spring或应用服务器这类的容器，容器可以集中或在外部配置数据源，然后放置一个JNDI上下文的引用 -->
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/yycg"/>
                <property name="username" value="root"/>
                <property name="password" value="941005"/>
                <!-- 默认连接事务隔离级别 <property name="defaultTransactionIsolationLevel" value=""   
                    /> -->
            </dataSource>
        </environment>
    </environments>
    <!-- 映射文件，mapper的配置文件 -->
    <mappers>
        <!--直接映射到相应的mapper文件-->
        <mapper resource="com/lzj/study/usermapper.xml"/>
        <mapper resource="com/lzj/study/blogmapper.xml"/>
        <!--扫描包路径下所有xxMapper.xml文件-->
        <!-- <package name="com.xhm.mapper"/> -->
    </mappers>
</configuration> 
```

#### 	usermapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.lzj.study.dao.UserMapper">
    <!-- 定义数据库字段与实体对象的映射关系  
  		resuleMap的作用
  		1，数据库的字段和类中属性做一一对应
  		2，查询时只会查询出在resultMap中已申明的属性-->  
    <resultMap id="userWithBlog" type="user">
        <id column="id" property="id"></id>
        <result property="name" column="name"></result>
        <result property="passward" column="passward"></result>
        <!--property表示等待注入的值，column的值今后会 传递给 resultSet.getString(columnName) 对于这种1-1模式可以不要-->
        <!--为什么加上这个元素的，mybatis会自动查询子表的内容
 			1，因为在查询语句中已经指定了两个表的存在-->
        <association property="blog" column="titleerere" resultMap="blog"></association>
    </resultMap>
    <resultMap id="blog" type="blog">
        <id column="id" property="id"></id>
        <result column="title" property="title"></result>
        <result column="content" property="content"></result>
    </resultMap>
    <!-- 根据id查询ticket, 关联将Customer查询出来 -->  
    <select id="findUserById" parameterType="int" resultMap="userWithBlog">
    	select
    	*
    	 from user u inner JOIN  blog b on b.user_id=u.id where b.user_id=#{id}
    </select>  
</mapper>
```
### 测试

```java
public class OneToOne {
	public static void main(String[] args) throws Exception{
		SqlSessionFactoryBuilder builder=new SqlSessionFactoryBuilder();
		ApplicationContext context=new ClassPathXmlApplicationContext();
		Resource resource=context.getResource("classpath:com/lzj/study/sqlConfig.xml");
		//factory中就已经解析了配置文件sqlConfig.xml中所有的属性并
		
		SqlSessionFactory factory=builder.build(resource.getInputStream(),"development1");
      //session 当中已经封装了解析xml的结果
		SqlSession session=factory.openSession();
		UserMapper userMapper=session.getMapper(UserMapper.class);
		User user=userMapper.findUserById(1);
		session.commit();
		System.out.println(user);
	}
}
```

