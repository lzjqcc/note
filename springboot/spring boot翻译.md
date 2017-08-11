spring boot学习

#### @EnableAutoConfiguration

这个是类级别的声明，这个注解是告诉Spring boot去根据你配置的jar包依赖去猜测你想要如何配置spring。这个自动配置。

Spring boot不推荐使用默认包，你可以使用@ComponentScan,@EntityScan or @SpringBootApplication 注解可以去扫描包中内容。

当你不需要某个类的配置可以使用@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})

#### @SpringBootApplication

这个注解相当于以下注解一起使用

```
@Configuration, @EnableAutoConfiguration and @ComponentScan
```

#### 主应用类的位置

通常建议Application类是放在其他类之上的，Application类上一般使用@EnableAutoConfiguration，因为这个注解能够在暗中去寻找包下确定的项。比如你在JPA类中声明了@Entity，这个@EnableAutoConfiguration到各个包下去寻找@Entity。

```
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
```

```java
//Application.class类上也会加上@Configuration这个注解
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

####  Configuration classes

通常建议这个@Configuration是放在主类中(包含main方法的类)。你能够使用@Import去添加configuration 类，你也可以选择使用@ComponentScan去自动扫描添加所有的spring 控件 当然也包括@Configurtaion类。如果要导入XML配置文件的话，你可以使用@ImportResource.

#### Spring boot 中属性文件

Spring boot属性文件默认文件名是application.properties

默认加载路径如下:

1. 当前目录下的/config目录
2. 当前目录
3. classpath下/config包
4. classpath root下

你可以使用spring.config.name来指定application.properites文件的名字。

​	使用spring.config.location来指定属性文件的路径	

### @EnableConfigurationProperties

给个实例然后解释下估计就明白了。在@EnableConfigurationProperties中注入的类可以在其他bean中使用这个就相当于 
使用@Component，@ConfigurationProperties(prefix="wisely2")在Wisely2Settings类上同时注解。
配置文件如下

```
    wisely2.name=wyf2  
    wisely2.gender=male2  
```
```java
    @ConfigurationProperties(prefix = "wisely2")  
    public class Wisely2Settings {  
        private String name;  
        private String gender;  
        public String getName() {  
            return name;  
        }  
        public void setName(String name) {  
            this.name = name;  
        }  
        public String getGender() {  
            return gender;  
        }  
        public void setGender(String gender) {  
            this.gender = gender;  
        }  
      
    }  
```
Application.class
```java
    @SpringBootApplication  
    //将Wisely2Setting.class注入
    	@EnableConfigurationProperties({Wisely2Settings.class})  
    public class DemoApplication {  
      
        public static void main(String[] args) {  
            SpringApplication.run(DemoApplication.class, args);  
        }  
    }  
```
然后就可以直接在其他bean中使用
```java
@Controller  
public class TestController { 
    @Autowired  
    Wisely2Settings wisely2Settings;  
  ... 
} 
```
#### Spring boot 整合Spring MVC
#####静态文件

静态文件：js ,image。。。这些文件默认是放在static 目录下，在网页中引用如下
```html
jquery-3.2.1.min.js的路径是：/src/main/resources/static/jquery3.2.1.main.js
<script src="/jquery-3.2.1.min.js"/>
```
#####.ftl文件
对于网页文件spring boot推荐使用 .ftl文件，这种类型的文件默认放在templates目录下
比如存放一个first.ftl文件
```java
/src/main/resources/templates/first.ftl
控制类
@Controller
public class HelloController {
    @Autowired
    @RequestMapping("/first")
    public String first(){
        return "first";
    }
}
//访问路径  http://localhost:8080/first
```
#####html
通过如下控制类访问html
```java
//如果要访问html一定要加上这个thymeleaf
@Controller
@RequestMapping("/thymeleaf")
public class HtmlController {
    //访问view.html页面
    @RequestMapping("/view")
    public String view(){
        return "view";
    }
    //访问hello.html页面
    @RequestMapping("/hi")
    public String hello(){
        return "hello";
    }
}
```
通过spring mvc访问html网页文件需谨记以下几点
1. index.html问默认访问页面。存放路径/src/main/resources/static/index.html  
  访问路径：http://localhost:8080/
2. 在pom.xml引入如下配置，不然访问页面会出现Circular view path : would dispatch back to the current handler URL again. Check your ViewResolver 错误。
```xml
    <!--导入这个-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
```
3. html页面要默认放在/src/main/resources/templates目录下。


