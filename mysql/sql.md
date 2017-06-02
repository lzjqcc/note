## 高效SQL

### 索引有效

name 字段中有三条数据:li zhang wang

​	

#### 进行后方一致/部分一致的检索的场合

​	如下模糊语句索引不会有作用：

```sql
无效：
select * from employee where name like '%w%' 部分一致
select * from employee where name like '%w'  后方一致
有效：
select * from employee where name like 'w%'
```

#### 使用了IS NOT NULL,[<>]比较运行符的场合(其他比较运算可以：如<=,>=)

​	这种场合索引不会有作用

```sql
无效：
select * from employee where name IS NOT NULL
select * from employee where name <> 'wang'
有效：
select * from employee where name='li' or name='zhang'
```

#### 对列使用的运算/函数的场合

​	

```sql
无效：
select * from employee where YEAR(birth)='1980' 判断员工出生年份是否是1980
有效：
select * from employee where birth>='1980-01-01' and birth<='1980-12-31'
```

#### 复合索引的第一列没有包含在where 条件中

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

​	