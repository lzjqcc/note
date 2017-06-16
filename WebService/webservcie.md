## Web Service

Web Service是用来解决数据孤岛问题，将隔离的系统中的数据进行整合

Web Service三元素：

> wsdl 文档说明语言
>
> uuid
>
> soap 封装HTTP协议，携带XML文件

![	webservice节点信息](/home/li/图片/笔记图片/webService/webservice节点信息.png)

访问webservice都要以POST请求

### JDK自带的与Apache CXF对比

#### JDK的自带的Web Service

> ​	类上需要使用@WebService注解
>
> ​	必须含有非static ,final的public方法
>
> ​	使用EndPoint.publish来启动服务

```java
@WebService
public class Service {
	public String sayHello(String name){
		System.out.println("hello "+name);
		return "hello "+name;
	}

	public static void main(String[] args) {
		String address="http://localhost:9090/hello";
		Endpoint.publish(address, new Service());
		System.out.println("complete");
	}

}
```

将wsdl变成Java文件(Java自带的工具旧版本1.0,1.1，不支持新版本)

wsimport -s . wsdl的网址(比如http://localhost:9090/hello)

#### CXF 

> 创建ServerFactoryBean并设置属性，bean.setAddress("http://localhost:9090/hello");	bean.setServiceClass(CXfDemo.class);
>
> 调用create方法启动服务

```java
package com.lzj.cxf;
//面向类
import org.apache.cxf.frontend.ServerFactoryBean;
import org.apache.cxf.tools.java2wsdl.processor.internal.ServiceBuilderFactory;
public class CXfDemo {
  	//没有public 方法并不会出错
  //注意参数一定不要是接口
	public void hello(String name){
		System.out.println("dd");
	}

	public static void main(String[] args) {
		
		ServerFactoryBean bean=new ServerFactoryBean();
		bean.setAddress("http://localhost:9090/hello");
		bean.setServiceClass(CXfDemo.class);
		bean.setServiceBean(new CXfDemo());
		bean.create();
	}

}
```

面向接口的代码如下

HelloService 实现了IHelloService

![面向接口](/home/li/图片/笔记图片/webService/面向接口.png)

```Java

```

当My Eclipse修改了Jdk的版本后需要修改的三个地方如下	![修改Java版本](/home/li/图片/笔记图片/webService/修改Java版本.png)

CXF自带的将wsdl转换为Java文件(这个是支持1.2新版的)

因为CXF自带的wsdl2java这个是shell小脚本 ，那就按小脚本的运行方式运行

```
sh wsdl2java -d /home/li http://localhost:9090/hello?wsdl
```
###### CXF与Spring 整合

###### ![选区_007](/home/li/图片/选区_007.png)

![选区_006](/home/li/图片/选区_006.png)

这个文件夹下有整合例子/home/li/下载/黑马视频/day68_webservice/资料/cxf3.0.2/apache-cxf-3.0.2/samples/java_first_spring_support/