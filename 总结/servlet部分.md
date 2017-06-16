### JSP

#### 静态include和动态include的区别

静态的inlcude会将目标页面的编译指令包含进来，动态的不会

#### 九个内置对象

request,response,session,application,config,out,exception,page(代表页面本身),

pagecontext（页面上下文，可以访问到页面中共享的数据，常用的方法有：getServletContext()和getServletConfig()）

## Servlet

#### Servlet和Jsp的区别

servlet侧重的是逻辑层，jsp侧重的是视图层

jsp的本质是servlet

#### 创建servlet的两个时机

- 客户端第一次请求servlet的时候
- web应用启动的时候立即创建servlet，load-on-startup(load-on-startup是一个整数值，整数值越小越先启动)

#### Servlet的生命周期

创建servlet实例

调用init()

调用service()

调用destory()

#### Filter 过滤器

作用：用于拦截客户的请求，拦截服务端的响应

#### 六个Listener

```
ServletContextListener:监听Web应用的启动和关闭
ServletContextAttributeListener:监听ServletContext(application)内属性的变化

ServletRequestListener:监听用户请求
ServletRequestAttributeListener:监听ServletRequest内的属性变化

HttpSessionListener:监听session的开始和结束
HttpSessionAttributeListener:监听HttpSession内的属性的变化
```

