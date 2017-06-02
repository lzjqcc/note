## 高效SQL

### 索引有效

name 字段中有三条数据:li zhang wang

​	

#### 	进行后方一致/部分一致的检索的场合

​		如下模糊语句索引不会有作用：

```sql
无效：
select * from employee where name like '%w%' 部分一致
select * from employee where name like '%w'  后方一致
有效：
select * from employee where name like 'w%'
```

#### 	使用了IS NOT NULL,[<>]比较运行符的场合(其他比较运算可以：如<=,>=)

​		这种场合索引不会有作用

```sql
无效：
select * from employee where name IS NOT NULL
select * from employee where name <> 'wang'
有效：
select * from employee where name='li' or name='zhang'
```

#### 	对列使用的运算/函数的场合

​	

```sql
无效：
select * from employee where YEAR(birth)='1980' 判断员工出生年份是否是1980
有效：
select * from employee where birth>='1980-01-01' and birth<='1980-12-31'
```

#### 	复合索引的第一列没有包含在where 条件中

```sql
创建复合索引：
create index index_name on employee (lname,fname)
无效：
select * from employee fname='xiao'
select * from employee lname='wang' or fanme='xiao' //这条语句因为检索条件的后面半句是对fname单独检索，所以无效
有效：
select * from employee lname='wang'
select * from employee lname='wang' and fname='xiao'
```

### 多表连接

​	多表连接时**子查询速度**比**连接查询慢**

​	订单基本信息表**order_basic**,

​	订单明细表**order_detail**

​	,产品表**product,**

​	用户表**user**

```sql
select ob.oid,ob.odate,p.pname,p.price,od.quantity,u.name
	from 
((order_basic ob inner join order_detail od on ob.oid=od.oid) inner join product p on p.id=od.id
  ) inner join user u on ob.uid=u.uid;
使用inner join .........on 连接起来两个或多个表，然后将其作为新表与其他表进行连接
```

#### 	使用视图

​		当存在**多次连接表查询**时使用到的**数据**都是来自**那几个相同的表**，这时可以考虑使用视图让复杂的语句简单化。

###### 	视图创建或删除语句

创建视图时select语句中不能包含一下内容：

- 系统变量/用户变量的参照表
- TEMPORARY 类型的表
- *from 语句后的子查询*				

```sql
create [or replace] view 视图名(列名,............) as select 语句 [with check option]
	with check option 表示不能插入或更新不符合视图的检索条件的数据（select 语句后面的where 条件）
drop view 视图名;
```

**注意**：

- ​	向实体表中插入或更新数据后，接着查询视图，数据是可以检索出来

  对视图进行**插入/更新/删除操作**时，需注意以下**限制**

  > 1，视图的列中含有统计函数的情况下，不能执行以上操作

  > 2，视图中使用了GROUP BY/HAVING 语句，DISTINCT语句，UNION语句的情况下，不能执行以上操作

  > 3，不能跨域多个表进行数据的插入/删除/更新操作，

### 存储过程

#### 	存储过程创建

```sql
DELMITER //
create procedure 存储过程名 （参数种类 参数名称 数据类型,.......)
begin
	处理内容
end
//
DELMITER ;
##DELMITER是改变零时改变分隔符
```

​	eg:创建存储过程sp_search		

```sql
DELMITER //
create procedure sp_search (IN p_nam varchar(20),OUT p_cnt INT)
begin 
	if p_nam IS NULL OR p_nam ='' THEN
		select * from customer;
	else
		select * from customer where nam like p_nam;
	end if;
	select FOUND_ROWS() INTO p_cnt;
end
DELMITER ;
```

#### 	执行存储过程

```
call 存储过程名 (参数,...);
```

​	eg:调用sp_search	

```sql
call sp_search('王%',@num)
select @num;##显示变量num
```

#### 	定义变量

```sql
声明变量：declare 变量名 数据类型[初始值];
赋值：set 变量名=值;		
```

​	eg

```sql
DELMITER //
create procedure sp(IN p_depart INT)
begin
	declare temp CHAR(4);
	set temp='孩子';
end
DELMITER ;
```

#### 	基本结构语句

​	

```
if 条件 then 执行命令
else 条件 then 执行命令
end if;

while 条件 do
	执行命令
end while
```

#### 	删除存储过程

```sql
drop procedure 存储过程名
```