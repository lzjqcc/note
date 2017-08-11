### Spring boot读取属性文件

#### 自动读取

自动读取属性的格式见

http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html

#### 利用@Value读取

@Value("${[spring](http://lib.csdn.net/base/javaee).profiles.active}")

private String profileActive;------相当于把properties文件中的spring.profiles.active注入到变量profileActive中

#### 利用@ConfigurationProperties

通过 @ConfigurationProperties(prefix = “home”) 注解，将配置文件中以 home 前缀的属性值自动绑定到对应的字段中。同是用 @Component 作为 Bean 注入到 Spring 容器中。

```
test.url=.....
test.key=....
```

```java
@Component
@ConfigurationProperties(locations = "classpath:application.properties",prefix="test")
public class TestProperties {
String url;
String key;

}
其他类中使用时，就可以直接注入该TestProperties 进行访问相关的值
```

#### 利用Environment类



```java
@Autowired
private Environment environment;
 .....
  environment.getProperty("test.url")
```