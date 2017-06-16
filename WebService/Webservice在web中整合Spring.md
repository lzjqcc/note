## Spring整合CXF

### Spring整合CXF伊始

新建WebService.java

> 一定要在类上加上@WebService注解

```java
package com.lzj.test;

import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebResult;
@javax.jws.WebService
public class WebService {
	@WebMethod(operationName="hello")
	@WebResult(name="retuernMsg")
	public String sayHello(@WebParam(name="name")String name){
		System.out.println("hello "+name);
		return "hello "+name;
	}
}

```

### Spring整合CXF第一步

​	在web.xml文件中添加如下配置

```xml
  <!--  配置Spring的监听器-->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!--  Spring配置文件的路径-->
    <context-param>
    	<param-name>contextConfigLocation</param-name>
    	<param-value>classpath:beans.xml</param-value>
    </context-param>
    <!--配置CXFServlet  -->
  <servlet>
  	<servlet-name>cxf</servlet-name>
  	<servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
  	<!-- CXF配置文件的路径 -->
  	<init-param>
  		 	<param-name>config-location</param-name>
    	<param-value>classpath:cxf-servlet.xml</param-value>
  	</init-param>
  </servlet>
  <servlet-mapping>
  	<servlet-name>cxf</servlet-name>
  	<url-pattern>/*</url-pattern>
  </servlet-mapping>
```

查看CXFServlet.java,可知CXFServlet.java默认是在/WEB-INF目录下寻找

```java
 //CXFServlet.java
String configLocation = servletConfig.getInitParameter("config-location");
        if (configLocation == null) {
            try {
                InputStream is = servletConfig.getServletContext().getResourceAsStream("/WEB-INF/cxf-servlet.xml");
                if (is != null && is.available() > 0) {
                    is.close();
                    configLocation = "/WEB-INF/cxf-servlet.xml";
                }
            } 
```

### Spring整合CXF第二步

​	在src下新建beans.xml和cxf-servlet.xml文件：

​	cxf-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws" xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
    <import resource="classpath:META-INF/cxf/cxf.xml"/>
    <import resource="classpath:META-INF/cxf/cxf-servlet.xml"/>
  <!--address 是访问的路径 -->
  <!-- #webService是引用beans.xml中bean webService-->
 <jaxws:endpoint address="/hello" implementor="#webService" ></jaxws:endpoint>
</beans>
```

​	beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
			http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
			http://www.springframework.org/schema/mvc 
			http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd 
			http://www.springframework.org/schema/context 
			http://www.springframework.org/schema/context/spring-context-3.0.xsd 
			http://www.springframework.org/schema/aop 
			http://www.springframework.org/schema/aop/spring-aop-3.0.xsd 
			http://www.springframework.org/schema/tx 
			http://www.springframework.org/schema/tx/spring-tx-3.0.xsd ">
  		<!--将WebService配置在Spring容器中 -->
			<bean id="webService" class="com.lzj.test.WebService"></bean>
</beans>				
```

### 最后

文件目录如下：

![选区_008](/home/li/图片/选区_008.png)	

结果：

![选区_009](/home/li/图片/选区_009.png)