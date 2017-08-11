### 自定义登录认证和权限认证

##### 登录

登录认证首先得指定登录页面

```
<form-login  login-page="/login.jsp"></form-login>
```

执行登录操作的时候主要重写一下几个类

```
1，AbstractAuthenticationProcessingFilter或者继承其子类UsernamePasswordAuthenticationFilter 用于拦截
2，UsernamePasswordAuthenticationToken 用于保存用户输入的数据。然后一般用传送到AbstractUserDetailsAuthenticationProvider中的authenticate中，然后UserDetails中保存的数据进行比较。
3，AbstractUserDetailsAuthenticationProvider 用于验证用户输入与数据库中的数据是否匹配。即登录验证操作。
4，UserDetails        保存用户权限以及你登录时要验证的数据
5，UserDetailsServuce  一般在loadUserByUsername中将登录用户与权限的关系封装成UserDetails对象
```

流程

```hxml
AbstractAuthenticationProcessingFilter拦截用户登录在attemptAuthentication方法中将用户信息封装成UsernamePasswordAuthenticationToken对象，
然后将UsernamePasswordAuthenticationToken委托给ProviderManager中AbstractUserDetailsAuthenticationProvider中的authenticate进行认证。
验证失败通过 throw new AccountExpiredException("用户名或密码或邮箱错误");在login.jsp页面中${SPRING_SECURITY_LAST_EXCEPTION.message}就可以在页面显示提示，指定登录失败的页面请看下面代码。
验证成功则创建Authentication对象返回。
```

```java
public class MyAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (true && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }
        String email=obtainemail(request);
        //将请求参数封装成toke 令牌
        UsernamePasswordEmialAuthenticationToken token=new UsernamePasswordEmialAuthenticationToken(obtainUsername(request),obtainPassword(request),email);

        this.setDetails(request,token);
        Authentication authentication=null;
        try {
            //登录失败捕获异常
            authentication = this.getAuthenticationManager().authenticate(token);
        }catch (AccountExpiredException e){
            //登录失败后重定向的页面。
            SimpleUrlAuthenticationFailureHandler failureHandler=new SimpleUrlAuthenticationFailureHandler("/login.jsp");
            try {
                failureHandler.onAuthenticationFailure(request,response,e);
            } catch (IOException e1) {
                e1.printStackTrace();
            } catch (ServletException e1) {
                e1.printStackTrace();
            }
        }
        return authentication;
    }
}
```

##### 资源访问的权限控制

 

```
1，AccessDesisionManager  登录成功的用户权限与访问url需要的权限进行对比
2
3，AbstractSecurityInterceptor 用于控制用户访问，主要是在方法beforeInvocation中借助FilterInvocationSecurityMetadataSource获取访问url所拥有的权限 和AccessDecisionManager的decide方法（用于比较用户拥有的权限与访问url所需的权限对比）。有权限不做处理，没有权限throw new AccessDeniedException("no right");指定的403页面。
4，FilterInvocationSecurityMetadataSource 将url与权限关联起来放入集合中
```

```java
//用户权限与访问url需要的权限进行对比
public class MyAccessDesisionManager implements AccessDecisionManager {
    /**
     *
     * @param authentication 用户权限
     * @param object 访问的url
     * @param configAttributes 访问该url应拥有的权限
     * @throws AccessDeniedException 用于显示没有权限的页面
     */
    public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
            if (object==null){
                return;
            }
            Collection<GrantedAuthority> authorityCollection= (Collection<GrantedAuthority>) authentication.getAuthorities();
            Iterator<GrantedAuthority> authorityIterator=authorityCollection.iterator();
            Iterator<ConfigAttribute> iterator=configAttributes.iterator();
            boolean flag=false;
            while (authorityIterator.hasNext()){
                GrantedAuthority authority=authorityIterator.next();
                while (iterator.hasNext()){
                    ConfigAttribute configAttribute=iterator.next();
                    if (configAttribute.getAttribute().equals(authority.getAuthority())){
                        flag=true;
                        break;
                    }
                }
            }
            if (!flag){
                //显示没有权限的页面
                throw new AccessDeniedException("no right");
            }
    }
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }
    public boolean supports(Class<?> clazz) {
        return true;
    }
}

```

配置文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<b:beans xmlns="http://www.springframework.org/schema/security"
         xmlns:b="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                        http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-4.0.xsd
                        http://www.springframework.org/schema/aop
                        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    <http pattern="/login.jsp" security="none"></http>
    <http pattern="/error.jsp" security="none"/>
    <b:bean id="detailsService" class="com.lzj.util.MyUserDetailsService"></b:bean>
    <http auto-config="true">
        <custom-filter ref="security" before="FILTER_SECURITY_INTERCEPTOR"></custom-filter>
        <custom-filter ref="authenticationFilter" after="SECURITY_CONTEXT_FILTER"></custom-filter>
        <!--指定登录页面-->
        <form-login  login-page="/login.jsp"></form-login>
        <csrf disabled="true"/>
        <!--配置403显示的也面-->
        <access-denied-handler error-page="/error.jsp"/>
    </http>
    <authentication-manager>
        <authentication-provider user-service-ref="detailsService">
        </authentication-provider>
    </authentication-manager>
    <b:bean id="security" class="com.lzj.util.MySecurityInterceptor">
        <b:property name="securityMetadataSource" ref="securityMetadataSource"></b:property>
        <b:property name="accessDecisionManager" ref="accessDecisionManager"></b:property>
        <b:property name="authenticationManager" ref="authenticationManager"></b:property>
    </b:bean>
    <b:bean id="securityMetadataSource" class="com.lzj.util.MyMetadataSource">
    </b:bean>
    <b:bean id="accessDecisionManager" class="com.lzj.util.MyAccessDesisionManager" >
    </b:bean>
    <b:bean id="authenticationManager" class="org.springframework.security.authentication.ProviderManager">
        <b:constructor-arg name="providers" >
            <b:list>
               <b:ref bean="provider"></b:ref>
            </b:list>
        </b:constructor-arg>
    </b:bean>
    <b:bean id="provider" class="com.lzj.util.MyDaoAuthenticationProvider" >
        <b:property name="userDetailsService" ref="detailsService"></b:property>
    </b:bean>
    <b:bean id="authenticationFilter" class="com.lzj.util.MyAuthenticationFilter">
        <b:property name="authenticationManager" ref="authenticationManager"/>
    </b:bean>
</b:beans>
```