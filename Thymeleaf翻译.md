##Thymeleaf学习
Thymeleaf是Spring推荐替代jsp的模板引擎
支持Spring Expression Language or SpringEL
Thymeleaf与Spring MVC集成的非常好    如下例子
在jsp当中你想要使用Spring MVC提供的标签，你的标签应该如下
```html
<form:inputText name="userName" value="${user.name}" />
```
在Thymeleaf提供的环境中，如下标签可以替代上面的标签
```html
<input type="text" name="userName" value="James Carrot" th:value="${user.name}" />
```
###开始
在使用Thymeleaf提供的方言之前你需要在html文件中引入如下配置
```html
<html xmlns:th="http://www.thymeleaf.org">
```
####使用th:text调用.properties文件中的值
这里#称为 messages表达式
在`WEB-INF`目录下新建`templates`子目录并存放`home.html`文件：
`/WEB-INF/templates/home.html`
```html
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
```
其中需要指明以下几点
1. 你需要在相同的文件夹(templates)下存放`.properties`文件

2. `.properties`文件名以home开头，如 home.properties,home_en.properties(对应English).......

3. 获取数据是靠键获取所以你要提供对应的键值对。

  这里的.properties文件对应的文件名，路径，内容如下
```
	文件路径及路径
    /WEB-INF/templates/home_en.properties for English texts.
    /WEB-INF/templates/home_es.properties for Spanish language texts.
    /WEB-INF/templates/home_pt_BR.properties for Portuguese (Brazil) language texts.
    /WEB-INF/templates/home.properties for default texts (if the locale is not matched).
```
```properties
#文件内容
home.welcome=¡Bienvenido a nuestra tienda de comestibles!
```
如果你想要从数据库中获取数据信息，你自己得实现`ImessageResolver`这个接口
###获取Request Session ServletContext中的数据

```
${x} will return a variable x stored into the Thymeleaf context or as a request attribute.
${param.x} will return a request parameter called x (which might be 	  multivalued).
${session.x} will return a session attribute called x.
${application.x} will return a servlet context attribute called x.
```
###变量表达式
变量表达式指$ （OGNL表达式一致），不过获取的数据都是存在context variables(OGNL存放的数据还有root)
比如：${user.name} 指执行user对象的getName()方法
###标准表达式语法
标准的表达式语法指：Thymeleaf Standard Expression synatx
比如上面的两种：message表达式 ，变量表达式
```html
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
<p>Today is: <span th:text="${today}">13 february 2011</span></p>
```
这里还有许多表达式，如下
```
    Simple expressions:
        Variable Expressions: ${...}
        Selection Variable Expressions: *{...}
        Message Expressions: #{...}
        Link URL Expressions: @{...}
        Fragment Expressions: ~{...}
    Literals
        Text literals: 'one text', 'Another one!',…
        Number literals: 0, 34, 3.0, 12.3,…
        Boolean literals: true, false
        Null literal: null
        Literal tokens: one, sometext, main,…
    Text operations:
        String concatenation: +
        Literal substitutions: |The name is ${name}|
    Arithmetic operations:
        Binary operators: +, -, *, /, %
        Minus sign (unary operator): -
    Boolean operations:
        Binary operators: and, or
        Boolean negation (unary operator): !, not
    Comparisons and equality:
        Comparators: >, <, >=, <= (gt, lt, ge, le)
        Equality operators: ==, != (eq, ne)
    Conditional operators:
        If-then: (if) ? (then)
        If-then-else: (if) ? (then) : (else)
        Default: (value) ?: (defaultvalue)
    Special tokens:
        No-Operation: _
```
这里所有的表达式都可以嵌套，组合使用。
####Message表达式
上面的那种方式缺陷不能动态的获取信息
在此做修改，在首页根据登录，动态的获取用户名并使用message表达式将用户名放到页面上。
properties文件的内容应如下
```properties
home.welcome=¡Bienvenido a nuestra tienda de comestibles, {0}!
```
html文件 使用组合方式
```html
<p th:utext="#{home.welcome(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```
其实这个message key也可以动态的获取
```html
<p th:utext="#{${welcomeMsgKey}(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```
这里列举一些常用的OGNL表达式
```
获取list 数组中的内容
${personsArray[0].name}
获取Map中的内容
${personsByName['li'].age}---》personByName.get("li").getAge()
/*
 * Methods can be called, even with arguments.
 */
${person.createCompleteName()}
${person.createCompleteNameWithSeparator('-')}
```
####内置对象
Thymeleafe提供了许多内置对象，内置对象的调用语法如下

`${#内置对象}`
```

    #execInfo: information about the template being processed.
    #messages: methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
    #uris: methods for escaping parts of URLs/URIs
    #conversions: methods for executing the configured conversion service (if any).
    #dates: methods for java.util.Date objects: formatting, component extraction, etc.
    #calendars: analogous to #dates, but for java.util.Calendar objects.
    #numbers: methods for formatting numeric objects.
    #strings: methods for String objects: contains, startsWith, prepending/appending, etc.
    #objects: methods for objects in general.
    #bools: methods for boolean evaluation.
    #arrays: methods for arrays.
    #lists: methods for lists.
    #sets: methods for sets.
    #maps: methods for maps.
    #aggregates: methods for creating aggregates on arrays or collections.
    #ids: methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
......
```
####Expressions on selections (asterisk syntax)
```html
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
  </div>
```
使用*号表达式，上面相当于下面的内容
```html
<div>
  <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>
```
注意如果没有th:object 话$和*效果是一样的
####Link URLS

```html
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" 
 th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```
注意：
 对于相对的url 将会自动的添加项目名称

####Fragments
详见 http://schy-hqh.iteye.com/blog/1961397
####boolean null 
null
```html
<div th:if="${variable.something} == null"> ..
```
boolean
```html
<div th:if="${user.isAdmin() == false}"> ...
```
####Appending texts
在字符后面添加新字符直接在后面用+
```html
<span th:text="'The name of the user is ' + ${user.name}">
```
###学习资源
http://www.jianshu.com/p/a7056b023df0   中文翻译
http://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#literal-tokens 官网介绍
