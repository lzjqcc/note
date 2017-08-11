spring security 认证源码分析

#### 登录

spring security 认证是依靠AbstractAuthenticationProcessingFilter 这个类来完成的，首先看doFilter方法

```java
。。。。
authResult = attemptAuthentication(request, response);//这个是抽象方法，因此查找子类的实现
。。。。
```

   AbstractAuthenticationProcessingFilter 子类是 UsernamePasswordAuthenticationFilter查看attemptAuthentication方法的实现

```java
public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
  		//如果不是post请求直接抛出异常
		if (postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}
		//获取request中用户名
		String username = obtainUsername(request);
  		//获取request中密码
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();

		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password);

		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);

		return this.getAuthenticationManager().authenticate(authRequest);
	}
```



```java
//usernameParameter 值为username 
protected String obtainUsername(HttpServletRequest request) {
		return request.getParameter(usernameParameter);
	}
//passwordParameter 值为password
protected String obtainPassword(HttpServletRequest request) {
		return request.getParameter(passwordParameter);
	}
```

通过上述代码分析可知：如果想自定义表单的话，表单应该满足一下

> 请求方法为post
>
> input 中 用户名中name默认是 username,密码中name默认是password

#### 资源控制





```java
AbstractSecurityInterceptor//用于拦截url  默认实现：FilterSecurityInterecptor
  		
  //用于封装访问控制的pattern0 = {LinkedHashMap$Entry@5663} "Ant [pattern='/user/**']" -> " size = 1"
//                          1 = {LinkedHashMap$Entry@5664} "Ant [pattern='/admin/**']" -> " size = 1"
  	FilterInvocationSecurityMetadataSource FilterInvocationSecurityMetadataSource 		
 		 //用于获取权限
		private Authentication authenticateIfRequired() {
		Authentication authentication = SecurityContextHolder.getContext()
				.getAuthentication();

		if (authentication.isAuthenticated() && !alwaysReauthenticate) {
			if (logger.isDebugEnabled()) {
				logger.debug("Previously Authenticated: " + authentication);
			}

			return authentication;
		}
  
  
AbstractAccessDecisionManager //控制访问url认证
  	//这个方法主要用于用户拥有权限和url访问权限进行认证
  //其中Authentication 封装org.springframework.security.core.userdetails.User@d7d: Username: li; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: ROLE_USER
	
  
  decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes)authentication:为用户拥有权限 object:url  configAttributes:包含访问url所拥有的权限   用户认证是否有权限通过   
  默认是使用AffirmativeBased decide底层的认证是通过AccessDecisionVOter的子类实现的，返回的result 0 1 -1
  	switch (result) {
			case AccessDecisionVoter.ACCESS_GRANTED://1
				return;

			case AccessDecisionVoter.ACCESS_DENIED://-1
				deny++;

				break;

			default:
				break;
			}
```

