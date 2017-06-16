## mybatis分页

Mybatis使用RowBounds对象进行分页，这是针对ResultSet结果集执行的内存分页，不是物理分页。

### 第一种分页

#### 	userMapper.xml

> bind 中value可以执行算数运算
>
> bind 中是根据对象的属性的getter setter方法来获取获取或注入值，没有这两个方法就会报错

```xml
    <select id="findOnlyUserByPage" parameterType="com.lzj.study.dao.LimitCondition" resultType="user">
        <if test="firstCondition==1">
          <!-- 注意-->
            <!--bind 中value是可以进行算数运算，value的取值是根据对象的getter 和setter方法来的 -->
              <!--firsCondition在LimitCOndition中一定要含有setter getter方法 --></-->
              <--  --></-->
          <bind name="pageFirst" value="(firstCondition-1)*2"></bind>
        </if>
        <if test="firstCondition!=1">
            <bind name="pageFirst" value="firstCondition*2-1"></bind>
        </if>
        select * from user where id>=#{pageFirst} order by id limit 2;
    </select>
```

反例：bind 

```xml
  <select id="findOnlyUserByUid" parameterType="Integer" resultType="user">
    <!--Integer 中没有first的getter setter方法，所以报错 -->
       <bind name="first" value="first"></bind>
    select * from user where id>=#{first}*2 ORDER  by id limit 2;
    </select>
```

### 第二种分页

​	userMapper.xml

> 通过between来进行分页

```xml
  <select id="findOnlyUserByLimitCondition" parameterType="com.lzj.study.dao.LimitCondition" resultType="user">
        <bind name="pageFirst" value="firstCondition"></bind>
        <bind name="pageSecond" value="secondCondition"></bind>
        select * from user where id between #{pageFirst} and #{pageSecond};
    </select>
```

### 第三种插件分页