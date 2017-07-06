## Spring boot 学习

常用注解  http://blog.csdn.net/lafengwnagzi/article/details/53034369

### 理解

在pom.xml中通过添加相应的starter模块（就是spring boot的依赖），来添加相应的依赖包。

### 第一个Hello Word

##### 	第一步

​	在Eclipse中新建一个maven Java项目

##### 	第二步

​	在pom.xml文件中加入如下配置

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.4.0.RELEASE</version>
</parent>
	
spring-boot-starter-web  包含了很多内容，spring-webmvc、spring-web、jackson、validation、tomcat、starter。
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
</dependency>
	
```

##### 	第三步

新建一个Controller用户返回接受请求

```java
//@RestController @RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。（返回的数据就是一个json数据）
@RestController
public class App {
	private static List<Person> list=new ArrayList<Person>();
	static{
		list.add(new Person("li", "20"));
		list.add(new Person("hao", "30"));
	}
	@RequestMapping("/hello")
	public List<Person> sayHello() {
		return list;
	}
	
}
```

##### 	第四步

新建一个springboot应用，并点击右键运行

```java
//标注这是一个 springboot应用的标识
//等价于以默认属性使用 @Configuration ， @EnableAutoConfiguration 和 @ComponentScan
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		//具体run方法会启动嵌入式的Tomcat并初始化Spring环境及其各Spring组
		SpringApplication.run(Application.class, args);
	}

}
```

##### 最后

在地址栏中输入：

[](http://localhost:8080/hello)



![springboot](D:\笔记\springboot.PNG)