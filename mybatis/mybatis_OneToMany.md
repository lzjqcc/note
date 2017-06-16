## Mybatis One To Many

Mybatis One To Many 和One To One在Idea中mybatisgenerator    com.lzj.mybatis.study包下

### 配置文件

#### usermapper.xml

```xml
 <resultMap id="userWithManyBlog" type="user">
        <id column="id" property="id"></id>
        <result property="name" column="name"></result>
        <result property="passward" column="passward"></result>
        <collection property="blogList" column="user_id" resultMap="blog" ofType="blog"></collection>
    </resultMap>
    <resultMap id="blog" type="blog">
        <id column="bid" property="id"></id>
        <result column="b_title" property="title"></result>
        <result column="b_content" property="content"></result>
    </resultMap>
    <!--一个用户多个博客:注意当blog 主键为id , user 主键也为id时，如果不对其中的一个主键改名则1-多查询结果只有一个-->
    <select id="findUserWithManyBlogByUid" parameterType="int" resultMap="userWithManyBlog">
       select
       u.id,
      b.id as bid,
      b.title as b_title,
      b.content as b_content
        from user u inner JOIN  blog b on b.user_id=u.id where u.id=#{id}
    </select>
```

其他配置文件见mybatis OneToOne

#### 测试

```java
	public static void main(String[] args) throws Exception{
		SqlSessionFactoryBuilder builder=new SqlSessionFactoryBuilder();
		ApplicationContext context=new ClassPathXmlApplicationContext();
		Resource resource=context.getResource("classpath:com/lzj/study/sqlConfig.xml");
		//factory中就已经解析了配置文件sqlConfig.xml中所有的属性并
		// size = 6
		SqlSessionFactory factory=builder.build(resource.getInputStream(),"development1");
		SqlSession session=factory.openSession();
		UserMapper userMapper=session.getMapper(UserMapper.class);
		//一个用户一个博客
	//	User user=userMapper.findUserById(1);
		//一个博客多个用户
		User user=userMapper.findUserWithManyBlogByUid(1);
		List<Blog> list=user.getBlogList();
		session.commit();
		System.out.println(list);
		System.out.println(list.size());
	}
```