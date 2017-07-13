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