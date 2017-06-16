### Struts2中属性驱动和模型驱动的区别

- 模型驱动的Action要实现ModelDriven接口，而且提供相应的类型，不需要为此类型提供setter getter方法。
- 属性驱动需要提供属性的getter setter方法，如果是复杂类型（OGNL类型转换器）就要提供无参构造方法。

### Action访问Servlet API

1,ActionContext 

2，实现ServletContextAware ,ServletRequestAware,ServletResponceAware接口

3，ServletActionContext中含有静态方法可以访问servlet中的api

### 

### Stack Context 中根(ValuesStack)对象 和普通对象的区别

1，普通对象需要在对象名之前添加#前缀

2，根对象可以省略对象名

### 如何将一个对象放入StackContext中和ValueStack中

​	1，ActionContext.getContext().put("bb", "bb");   //放入StackContext

​	2，Action对象默认是在ValueStack中

```
比如Action中含有一个属性是
private Person person=new Person();
在jsp中显示person中某个属性的值就是：person.age  因为这时候action的对象名已经省略。
```

​	3，放入valueStack中ActionContext.getContext().getValueStack().push(person);