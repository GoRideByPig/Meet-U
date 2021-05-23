# SQL语言



## Constraints（限制）

1. **外键引用表和主表同时删除和更新：**

```sql
constraint chinese_blocks_fk foreign key(movieid,rn)
	references chinese_titles (movieid,rn) on delete/update cascade
```

这样设置以后，当表`chinese_titles`删除或者更新的时候，这个引用表中所有匹配`movieid,rn`的记录也会同步删除或者更新。



## Date（日期）



1. **日期格式是多样的：**不同国家的日期格式都不相同，有一点很迷惑，欧洲的日期格式为`dd-MM-yyyy`，而美国的格式为`MM-dd-yyyy`有时候会分不清日和月。
2. **将日期格式字符串转换成日期：**`date('2018-03-12')`,可以支持比较

```sql
where post_date >= date('2018-02-11')
```

3. **用户输入日期存储到数据库：**一定要用日期格式存储，不要用字符串，这样数据库就没法用相关函数了。
4. **`datatime`数据类型（或者叫`timestamp`)：**注意，这个数据类型除了有==日期==之外，还有==时间==。
5. **`date`和`datetime`的类型转换问题：**如果要把一个`date`转换成`datetime`，那么对应的时间时`00:00:00`
6. **`date`数据类型运算：**

```sql
select date('2021-05-30') + interval '1 month';
```

7. **取出对应日期相关的日期属性：**`date_part('year',now())`,其中的`now()`是`datetime`格式，它指的是现在的日期时间。



## Transaction（事务）

1. 在现实世界购物的时候，我们交易物品需要这样几个步骤：

   (1).店家要把货物给你准备好。

   (2).你要给店家付钱。

   (3).店家把货给你。

   这几个部分**缺一不可**，如果只完成了一部分，那就导致整个**事务**是不完整的。如果是**交易毒品**，**有人把💴交了，但是店家却没有把货给你，**那么这个交易就是失败的。

2. 还比如**转账**这个操作，我要把一个账户里的💴取出来，然后放到另一个账户上去，**这个过程必须是一体化的，要么全部完成，要么全部完不成。**

3. 在数据库中，会涉及到频繁的数据修改，我们可以把这些修改操作看成一个整体（**事务**)。用`begin transaction;`开启一个事务。然后在最后的时候，用`commit;`来结束事务**并提交操作。**`rollback;`是**回滚**操作，表示我要把这个事务的状态移动到事务开始时的状态。
4. ==注意：==很多数据库管理系统采用了`auto commit`状态，如果采用了`insert into...`修改了数据，而且没有`begin transaction;`的话 ，**那么修改会立刻生效**。

5. **锁的概念：**在`begin transaction;`命令执行之后，我这个数据库中的数据**就会被加上🔒，任何第三方都不允许修改我这个数据库了。**
6. **`DDL`操作支不支持事务？**在`SQLServer`和`Postgresql`中，DDL操作同样支持事务，但是在`mysql`中就不支持，`create,alter,drop`等操作即刻就会生效。

7. **实际的银行交易操作：**实际的银行操作不会把账户里的💴数减去

（1）首先，我要在账户表中**新增一列`Date`**，用于记录这个是哪一天的余额。

（2）之后，我会在一个**操作表**中记录一下这个转账操作。

（3）最后，如果要查看现在的余额，我就会用这两个表**做运算**得到最后的结果。

8. **视频的点赞👍机制：**如果我点了一下赞，我就会记录一个信息，这个信息中存了**视频的ID**和**用户的ID**，可能会有一个`Date`。**统计这个视频有多少赞的时候**，我就把视频的ID选出来，统计一下有多少条记录就可以了。

9. **网页上删除和数据库内的删除不是同步的：**在网页上只是==不可见==，数据库中会有一列用于记录**这个帖子到底是可见还是不可见。**在显示给网页的时候，只需要命令`select * from... where active = 'Yes'`

10. **事务的性质`ACID`:**

（1）**Atomicity（原子性）: **事务中的语句，要么全部执行完毕，要么整体全部不执行。

（2）**Consistency（一致性）：**事务开始和结束时，除了事务带来的改变，数据都要保持一致。

（3）**Isolation（隔离性）：**事务正确提交之前，不允许把事务对数据的改变提供给任何其他的事务，即事务正确提交之前，它可能的结果不应该显示给其他事务。

（4）**Durability（持久性）：**事务正确提交之后，其结果永远保存在数据库种，即使在事务提交之后有了其他的故障，事务的处理结果也会得到保障。

11. **并发事务下会出现的常见问题：**

（1）**脏读（Dirty Read）：**<font color=red>事务A能够读到事务B还没有提交的数据。</font>解决的核心是：==对于事务B没有提交的数据，事务A不能读到==

（2）**不可重复读（Nonrepeatable Read）:**<font color=red>事务A读了一个数据，之后这个数据在事务B中修改或删除并提交了，事务A再查一下这个数据发现不一样了。</font>

（3）**幻读（Phantom Read）:出现的前提是==并发的事务中有事务发生了插入或者删除操作==，**<font color=red>事务A修改了表中的所有数据，之后事务B又在这个表中插入了一条数据，然后提交了，然后事务A再查询发现我竟然有一条数据没有修改。</font>

12. **事务的隔离级别：**

* **Read-Uncommitted:** <font color=green>一个事务A能够读到另一个事务B修改了但没有提交的数据。</font>三种问题什么都没有解决，只能保证==持久性。==
* **Read-Committed:（语句级别）** <font color=green>一个事务A只能够读到事务B修改并提交的数据。</font>，可以防止脏读。
* **Repeatable read**:
* **Serializable:**<font color=green>最高的事务隔离级别，不管多少个事务，挨个运行完一个事务的所有子事务后，才可以运行另一个事务的所有子事务。</font>能够解决全部问题。

## Insert（插入记录）



1. **`insert`语句的基本语法规则：**

```sql
insert into 表的名字 (col1,col2,...coln) values (value1,value2,...valuen)
```

2. **语法变式1：后面`value`的列表可以有多个，分别用`,`隔开，实现用一条`insert`**语句插入多个数据。

```sql
insert into 表的名字 (col1,col2,...coln) values
(value1,value2,...valuen),
(valuep,valueq,...valuer)
```

3. **语法变式2：**如果语句后面没有`col`的列表，那么`value`列表中的值会对应地插入到第一列，第二列…

```sql
insert into 表的名字 values(value1,value2,...,valuen)
```

==一般不建议这样做，因为当你又插入或者删除一个新的列的时候，列的顺序可能会发生变化，这个时候插入的值可能就不一样了。==

4. **语法变式3：**如果列中间少一个

```sql
insert into 表的名字 (col1,col2,col4) values (val1,val2,val4)
```

**那第三列的值是多少？**

（1）首先，我会去看一下第三列的属性是不是又**默认值**(default value)，如果有，就设成默认值。

（2）如果没有默认值，这一列就会设置成`null`

（3）如果这一列有一个`constraint`是`not null`，那么就会报错。

5. **默认值的指定方法：**

```sql
create table 表的名字
(
    列的名字 列的数据类型 default 你设的默认值...
)
--举例
create table table_name
(
    id integer default 10 not null
);
```

（1）一般常用的默认值是`CURRENT_TIMESTAMP`,可以记下来**这条记录插入数据库的时间。**

6. **`insert`中的并发问题：**

（1）电影中有`movieid`这个属性，我现在要插入一条新的记录，我先用命令`select max(movieid)+1`获取到了当前电影的最大`id`，之后再把这个作为信息插入到数据库当中。

（2）==这样做是非常危险的==，**大多数时候数据库是很多人同时操作的，**如果我有两个人同时添加，那么就会有一个人的记录添加不上去。

7. **解决`insert`中并发问题的方法**：数据库支持`sequence`机制，本质上，这就是一个数值发生器，默认从$1$开始，一直可以到一个很大的值。使用方法如下：

```sql
--创建序列
create sequence movie_seq;
--Oracle
insert into movies(movieid,...) values (movie_seq.nextval,...)
--如果我再movies表中添加了这个信息，我还需要在credits表里面增加有关这个电影的附加信息，这个时候就要调用currval
insert into credits(movieid,...) values
(movie_seq.currval)

--Postgresql
insert into movies(movieid,...) values
(nextval('movie_seq'),...)
insert into credits(movieid,..) values
(currval('movie_seq'),...)
```

8. **设置自动增量的列：**在`create table`后面定义列的时候，在后面加一个`serial(postgresql中)`

```sql
create table table_name
(
    id serial not null
    --注意，如果用了serial以后，就不用定义数据类型了，因为我已经知道了
);
```

9. **我如果用了自动增量的列，我插入了一条数据，我怎么获取到我刚才插入的这条记录的那个列的值？**：在`postgresql`中，可以使用`lastval()`方法来获取。

```sql
create table test
(
   id serial,
   name varchar(10)
);

begin transaction;
    insert into test(name) values ('lee');
commit;

select lastval(); --1

insert into movies(title,...)
values('Some Movie Title')
insert into credits(movieid,..)
values(lastval(),...)
```





## Update（更新记录）



1. **基本语法规则：**`update 表的名字 set 列名 = 新的列值`,==后面一定要加上`where`==

```sql
update table_name set column = new_value
```

2. **语法变式1：一次性更新多列 **,`set`后面的那个`column=new_value`弄多个，每个之间用`,`隔开。

```sql
update table_name
set
col1 = val1,
col2 = val2,
...
```

3. **使用注意！**==用`update`时，先用`where`选出来这些行，看看这些行是不是符合题意！！==

4. **实例：德国人的贵族名**

（1）德国人的名字时`名+姓`的格式，在姓中，如果是贵族，会在原来的姓前面加上一个`von`，用来表示贵族。比如`John von Neumann`

（2）现在如果要把德国人**按照姓进行排序**，我需要把原来的姓放到前面。

（3）处理思路：

​	a.首先，利用**子串**，把后面的姓搞出来。

​	b.之后，在**子串**后面拼接上`‘von’`

```sql
update people
set surname = substr(surname,4) || '(von)'
where surname like 'von %';
```

5. **`update`操作的本质：**`update`是一个**集合操作**，它可以把`where`选出来的记录的集合统一进行操作，**注意不要把`update`放在循环里进行操作，应该先选出来所有的，然后用一条`update`进行操作。**
6. **主键锁死**：注意<font color=green>主键所涉及的属性是不能被修改的，如果真的要修改，唯一的办法就是删除这条记录，然后重新添加。</font>
7. **“更新还是插入？” 问题**

（1）我有一个表叫`movies`，现在我要再加两列`duration,color`，这两列的数据要从另外一个叫`us_movies_info`来导入。

（2）首先明确，**Primary key**是`title,country,year_released`.

（3）现在的问题是：**`us_movies_info`这个表中的信息，有的在原表中有，需要的是`update`，但是有的在原表中没有，需要的是`insert`，我怎么处理？**

==处理思路：==

**第一步**：**首先考虑需要`update`的情形**,也就是`us_movie_info`中没有新的电影的时候：

​	我首先把用`movies`中的信息，在`us_movie_info`表中把对应的信息找出来

​	找出`duration`：假设返回结果叫`duration_result`

```sql
select duration from us_movie_info i
	where i.title = movies.title
	and i.year_released = movies.year_released
```

​	找出`color`: 假设返回结果叫`color_result`

```sql
select case color
		when 'C' then 'Y'
		when 'B' then 'N'
		end as color
	from us_movie_info i
where i.title = movies.title
and i.year_released = movies.year_released
```

​	**问题1：由于`update`是集合操作，上述查询会对`movies`中的每一条记录都进行操作，如果我的`movies`中有了新的记录，但是在`us_movie_info`中却没有那怎么办？**

​	<font color=red>解决方法：用`exists`关键字，在`update`的`where`条件中先选出来看存不存在</font>

```sql
update movies
set duration = duration_result,
	color = color_result
where movies.country = 'us'
and exists(
    select null
    from us_movie_info i2
    where i2.title = movies.title
    	and i2.year_released = movies.year_released
)
```

​	**问题2：对于上面`exists`中的操作，我每次要修改`movies`中的信息，就要在`us_movie_info`中查询一次，效率很慢，怎么改进？**

​	<font color=red>解决方法：用类似于`join`的方法，先把这两个表`join`起来，然后用这个过滤条件去筛</font>

```sql
where movies.country = 'us'
and movies.title = i.title
and movies.year_released = i.year_released
```

​	**问题3：我的`movies`中的这一条记录，按照`title,country,year_released`去和`us_movie_info`去匹配，发现有多条记录符合，怎么处理？**

​	<font color=red>没有办法，设置好主键！</font>

**第二步，考虑要`insert`的情形：**

思路：（1）用`left outer join`把`us_movie_info`和`movies`连接起来，这样如果`m.movieid = null`，就选除了新的电影，之后对于这条新电影的记录，我再选出相应的属性进行插入就可以了。

```sql
insert into movies (title,year_release,country,duration,color)
select i.title,i.year_released,'us',i.duration,
	case i.color
		when 'C' then 'Y'
		when 'B' then 'N'
	end
from us_movie_info i 
	left outer join movies m
		on m.title = i.title
		and m.year_released = i.year_released
		and m.country = 'us'
where m.movieid is null
```



## Delete（删除记录）



1. **网站的删除机制：**

（1）我有一个`operational_table`,如果你要删除一条记录，那么我就在这个表中的一列中把`Y`（可见）设置成不可见`N`。

（2）**如果这个表太大了：**我就会根据**时间长短**，把经过时间很长的记录放到另一个表`history_table`中。

2. **基本语法：**`delete from table_name where...`

==注意：如果忘记了`where`，所有的记录都会被删除！！==

3. **`Delete`的效率问题：**真实执行`delete`的时候，在提交事务之前，数据库<font color=red>会把数据全部备份一遍</font>，然后再删除，这个效率非常低。
4. **`Truncate`高效率删除：**直接置空表，不接受任何事务的操作。效率远远高于`Delete`。

5. **`foreign key`的限制作用**：如果一个表**被其他的表用外键指向了**，那么这个表中的数据不能随便删除，用于保证数据的一致性和完整性。
6. **保证数据完整性的另一种方法：**<font color=blue>进行用户权限管理</font>



## Function (函数)



1. **需求实例：**我要把修改后德国人的姓名`John Neumann(von)`变成原来姓名的格式`John von Neumann`显示。

（1）==特判：==如果`first_name`为空，那么就把`first_name`不显示。

（2）如果`first_name`不为空，那么就这样连接：`first_name || ' '`

（3）之后，在`surname`中寻找`(`出现的位置，如果是$0$，**表明并没有括号**，直接显示`surname`。

（4）如果不是$0$,则需要用方法`substr(text,startpos,length)`把括号内的东西截取出来，然后和姓进行拼接。

2. **函数的定义语法：**

![数据库函数定义格式](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\数据库函数定义格式.png)

```sql
create or replace function 函数名字(参数1 参数1的数据类型,参数2 参数2的数据类型,...)
#注意是returns
returns 返回值的数据类型
language plpgsql
as $$
declare
...
	begin
		函数体
	end;
$$ 
language plpgsql;#注意有;
```

3. **上述需求的代码：**

```sql
create function full_name(p_fname varchar,p_sname varchar)
#注意是returns
returns varchar
as $$
    begin
        return case
            when p_fname is null then ''
            else p_fname || ' '
        end ||
        case position('(' in p_sname)
            when 0 then p_sname
            else trim(substr(p_sname,position('(' in 					p_sname)+1,position(')' in p_sname)-position('(' 				in p_sname)-1))
            ||' '
            ||trim(substr(p_sname,1,position('('in 						p_sname)-1))
        end;#这个;是return的;
    end
    $$ language plpgsql; #注意要有;
    
#调用函数
select full_name(first_name,surname);
```

4. **将查询返回的结果放到局部变量：**

```sql
select col1,col2...
into local_var1,local_var2,...
from...
```

​	**问题：如果有返回了很多行怎么办？**

​	<font color=green>可以利用游标`cursor`</font>

5. **问题：既然有了函数，那我可以用函数封装查询，这样做好吗？**

（1）比如，我写一个函数，根据`country_code`去查询`country_name`，这样就避免了`join`操作。

```sql
#Postgresql
create function country_name(p_code varchar)
returns countries.country_name%type #%type表示取类型，这个意思是返回和countries.country_name一样的数据类型
as
	#函数的局部变量
	v_name countries.country_name%type
begin
	select country_name
	into v_name
	from countries
	where country_code = p_code;
	return v_name;
end;
```

（2）==不要这样做，这样做效率非常糟糕，用`join`的时候，数据库有无数种方法加速这个过程。==

（3）<font color=red>优先使用SQL，如果不能用一行SQL解决，就问一问有经验的人。能不用PL/SQL函数扩展就不用。</font>

6. **函数的确定性：**数据库中有确定性函数和不确定性函数，关键在于给定相同的`input`是不是一直有相同的`output`，比如函数`datename(month,'1970-01-01)`,这个函数的返回值**与数据库的设置有关系。**
7. **常见的库函数：**

（1）**生成数字序列**：`generate_series`，参数为$l,r,step$，表示生成$[l,r]$区间范围内步长为$step$的数字序列。比如`generate_series(1,5,2)`就会返回`1,3,5`

举例，从今天开始，生成未来7天的日期：

```sql
select current_date + date_table.interval as dates
from generate_series(0,28,7) as date_table(interval)
#注意date_table(interval)表示这个表中的第一列叫interval
```

![未来7天日期](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\未来7天日期.png)

还比如：<font color=red>注意：时间戳字符串标准格式为`yyyy-MM-dd hh24:mi:ss`</font>

```sql
#注意这个时间是04，且::timestamp把字符串的日期格式转换成了时间戳的日期格式
select * from generate_series('2021-05-03 16:42'::timestamp,'2021-05-03 18:00'::timestamp,'25 minutes');
```

（2）**分割字符串：**`split_part('我要分割的字符串','我要按哪个字符分割',我要取第几个元素（默认从1开始）)`

```sql
select split_part('Feel relaxed',' ',1) #Feel
```

（3）**生成数字序列和分割字符串函数的应用：给字符串中的单词编号：**

第一步：把这个字符串转换成`text`格式，命名为`t1(words)`。

```sql
(select cast('Feel relaxed studying database' as text))
t1(words)
```

第二步：用上述字符串的整个字符的长度减去删去空格的长度，再加1就是单词数量`length(t1.words)-length(replace(t1.words,' ',''))+1`,之后生成数字序列，表示单词的编号。

第三步：把这个字符串和这个序列做一个笛卡尔积，最后用`split(t1.words,' ',t2.number)`选出单词就可以。

```sql
select split_part(t1.words,' ',t2.number)
from
(
    select cast('Feel relaxed about database' as text)
) t1(words)
cross join
(generate_series(1,length(t1.words)-length(replace(t1.words,' ',''))+1)) t2(number)
```

（4）**字符串子串：**`substr(字符串,l（默认起始为1）,s)`:从$l$开始（<font color=red>包含L</font>），向后面总共取$s$个字符。

应用：显示某个字符串中出现的每个字符

```sql
select distinct substr(t1.words,t2.number,1)
from
(select 'Feel relaxed about database'::text) as t1(words)
cross join
(generate_series(1,length(t1.words))) as t2(number);
```

（5）**显示某个字符的ASCII码**：`ascii(character)`，注意，如果里面接了一个字符串`ascii('abc')`，那么返回的是第一个字符的`ascii`码。

**库函数和触发器练习**

* **给定一个电影名称，要求查询有如下的结果：**
* ![练习1](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\练习1.png)

```sql
with t as (select cast('邋遢大王奇遇记' as varchar) title)
select title,substr(title,n,1) one_char,
		substr(title,n,2) two_chars,
		substr(title,n,3) three_chars
	from t
	cross join generate_series(1,200) n
where length(coalesce(substr(title,n,1)),'') > 0 order by n;
```

* **变式：要求求出来一个表，显示这个标题所有一个字符，两个字符，三个字符**

![练习2](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\练习2.png)

思路：把上面查出来的表和`generate_series(1,3)`进行笛卡尔积，之后在选择中用`case`，把不同的标题字符选出来

```sql
with t as (select cast('邋遢大王奇遇记' as varchar) as title)
select distinct case n
				when 1
					then one_char
				when 2
					then two_chars
				else three_chars
				end
				,n
from (select
     	substr(title,n,1) as one_char,
     	substr(title,n,2) as two_chars,
     	substr(title,n,3) as three_chars
     from t
     	cross join generate_series(1,200) n
     where length(coalesce(substr(title,n,1),''))  > 0 )x
     
     cross join generate_series(1,3) n order by n;
```

**然后把这个东西用函数封装：**

```sql
create or replace function chinese_split(p_chinese_text text)
    returns table(char_block text)
as $$
    begin
        return query
            with t as (select p_chinese_text as chinese_text)
            select distinct case n
                            when 1
                                then one_char
                            when 2
                                then two_chars
                            when 3
                                then three_chars
                            end
            from (
                select substr(chinese_text,n,1) as one_char,
                        substr(chinese_text,n,2) as two_chars,
                        substr(chinese_text,n,3) as three_chars
                from t
                    cross join generate_series(1,200) n
                where length(coalesce(substr(p_chinese_text,n,1),'')) > 0
)x
cross join generate_series(1,3) n;
end;
$$
language plpgsql;
```

**之后，创建一个新的表`chinese_blocks`，用来存储这些字符块**

```sql
create table chinese_blocks
(
    #注意严谨：名称，类型，限制之间的空格只能有一个！
movieid int not null,
rn int not null,
block varchar(3) not null,
constraint chinese_blocks_pk
primary key (block, movieid, rn),
constraint chinese_blocks_fk
foreign key (movieid, rn)
references chinese_titles (movieid, rn) on delete cascade
);
```

**之后，创建触发，当更新或者插入表`chinese_titles`的记录的时候，自动在`chinese_blocks`中做修改**,删除可以用`constraint`中的`on delete cascade`。

```sql
create or replace function chinese_title_split()
 returns trigger
as $$
begin
	if tg_op = 'update'
	then
		delete from chinese_blocks
		where movieid = old.movieid
			and rn = old.rn;
	end if;
	insert into chinese_blocks(movieid,rn,block)
		select
			new.movieid,
			new.rn,
			bl
			from chinese_split(new.title) as bl;
		return null;
	
	
end;
$$
language plpgsql;

#创建触发
create trigger chinese_titles_trg
after insert or update
on chinese_titles
for each row
execute procedure chinese_title_split();
```

**现在，我可以做到在电影中插入数据的同时，在`chinese_block`表中更新它的关键词。**

**接下来再创建一个函数，给定电影的任何局部信息，然后可以返回相关的电影id**

```sql
create or replace function chinese_candidates(p_user_input text)
returns table(movieid int)
as $$
	select movieid
	from (
        select movieid,rank() over (order by hits desc) rnk
        from (
            #返回和用户输入匹配最多的电影
            select cb.movieid,count(*) as hits
            	#然后把用户的输入分割用方法分割成块
            	from chinese_split(p_user_input) searched
            		join chinese_blocks cb
            			on cb.block = searched
            	#在chinese_blocks表中先按照电影id聚合一下
            	group by cb.movieid
        )x
    )y
    where rnk = 1;
$$
language sql;
```

**查询方法：**

```sql
select movieid,title from chinese_titles
where movieid in (select chinese_candidates('故事'));
```

**完善查询：通过其他表的`join`操作：**这个用于练习`join`，理解`join`。

```sql
select
title,

also_known_as,
#string_agg是聚合函数，将聚合的所有属性按照一个字符为连接符进行连接
string_agg(case c.credited_as when 'D' then trim(p.surname ||' ' ||coalesce(p.first_name,'')) else null end,',') directors,
#用于连接所有演员
string_agg(case c.credited_as when 'A' then trim(p.surname||' '||coalesce(p.first_name,'')) else null end,',') actors

from
(
    select
    cm.title,
    string_agg(at.title,',') also_know_as,
    co.country_name || ', '||m.year_released origin,
    m.movieid
    from
    (
        #在这个表中，我把和‘惠普凌河相关度最高的movieid表和chinese_title表做了连接，然后选出来排名第1（也就是简体中文的标题）的电影的电影id，标题
        select
        ct.movieid,
        ct.title,
        from chinese_candidates('惠普凌河') cc
        	join chinese_titles ct
        	on ct.movieid = cc.movieid
        	where ct.rn = 1
    ) cm
    #之后，我把这个表和movies表连接，用来扩展信息
    join movies m
    	on m.movieid = cm.movieid
    join countries co
    #与movies表连接后，我又可以和country表连接，用来扩展信息
    	on co.country_code = m.country
    left join alt_titles at
    #之后，我再和标题表title连接，筛选这个电影的别称
    	on at.movieid = cm.movieid and at.title <> cm.title
    group by cm.title,
    		co.country_name,
    		m.year_released,
    		m.movieid
) x
#我把这部电影的信息选完之后，我在和credits表进行连接用于扩展演员信息
	left join credits c
		on c.movieid = x.movieid
	left join people p
		on p.peopleid = c.peopleid
group by title,
		also_known_as,
		origin
order by origin;
    			
```

查询结果：

![查询结果](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\查询结果.png)

8. **函数中的条件语句：**

（1）**定义格式：**

```sql
begin
	if condition1
	then
		...
	elseif condition2
	then
		...
	else
		...
	end if;
end;
```

（2）**需求实例：根据不同地区来组合姓和名**

<font color=red>细节：处理每一个字段的时候，一定要考虑一下这个字段是否会有空值。还有，对于字符串别忘记最后要用`trim`去除空白字符</font>

```sql
create or replace function full_name(p_fn varchar,p_sn varchar,style char)

returns varchar

as $function$
begin
	if upper(style) = 'W'
	then
		return trim(coalesce(p_fn,'') || ' ' ||p_sn);
	elseif upper(style) = 'E'
	then
		return trim(p_sn || ' '||coalesce(p_fn,''));
	else
		raise exception 'Style must be W or E';
	endif
end;
$function$
language plpgsql;
```

9. **函数中的循环**: 

（1）定义格式：

```sql
for 变量名 in 开始值..结束值 (这个是个闭区间)
	statements;
end loop;

while condition loop
	statements;
end loop;
```

（2）实例：计算阶乘：

```sql
create or replace function factorial(number int)
	returns int

as $function$
declare result int;
begin
	result := 1;
	for i in 1..number loop
		result = result * i;
	end loop;
	return result;
end;
$function$
language plpgsql;
```

或者用`while`循环写

```sql
create or replace function factorial(number int)
returns int

as $function$
declare result := 1
		i := 1
begin
	while i <= number loop
		result = result * i;
		i = i + 1;
	end loop;
	return result;
end;
$function$
language plpgsql;
```

10. **函数返回表的方法：**

```sql
create or replace function aaa(...)
returns
	table(
        col_name1 col_type,
        col_name2 col_type #这里写的列要和后面查询出来的列对齐.
    )
   
    as
    $$
    begin
    	return query select col1,col2 from ...;
    end;
    $$
    language plpgsql;
```

**实例：设计一个函数`character_table(string)`用来把一个字符串转换成一个表，表中含有每个字符和其对应的ascii码**

```sql
create or replace function character_table(pattern varchar)
	returns table
		(
            chr char,
            ascii int
        )

as
$body$
	begin
		return query
			select distinct (substr(t1.title,t2.index,1)::char) chr, ascii(substr(t1.title,t2.index,1)) ascii
			from (select pattern) t1(title)
			cross join generate_series(1,length(pattern)) t2(index)
			order by ascii;
	end;
$body$
language plpgsql;
```

11. **函数返回游标的方法：**

（1）**语法格式：**

```sql
create or replace function fun_name(arg1 type1,...)
returns 
	refcursor #背下来，返回游标就写这个单词

as
$$
declare
	ref refcursor; #背下来，这里也是这么写
begin
	open ref for
		#这里写查询
		select col1,col2 from table...;
	return ref;
end;
$$
language plpgsql;
```

（2）**如果需要返回的结果是很多表组成的结果集**,用`SETOF refcursor`

和`return next`来写

```sql
create function fun_name(arg1 type1,...)
	returns
		#背
		SETOF refcursor
	
	as
	$$
	declare
		ref1 refcursor;
		ref2 refcursor;
	begin
		open ref1 for select col1,col2 from table...;
		#背
		return next ref1;
		open ref2 for select col1,col2 from table...;
		return next ref2;
	end;
	$$
language plpgsql;
```

（3）**使用游标的方式：**==游标的生命在事务结束后就没有了==

第一步，开启事务

`begin transaction;`

第二步，调用函数，会返回一个游标的名称

`select character_cursor('I love database');`

第三步，用`fetch all in "游标名称"`获取结果，**注意用双引号。**

第四步，结束事务

`commit;`

12. **函数练习：**

（1）给定一个电影的`title`，请你识别出这个`title`是用哪种语言写的？

（2）**思路：**

​	a. 首先用求出来这个`title`中的每一个字符，和其对应的`ascii`码，然后选出<font color=red>其中有效字符</font>的码的最大值和最小值，之后用最大值和最小值判断这个语言所在的区间。

（3）**实现**：

```sql
create function script(fm varchar)
returns varchar
as $$
declare
	asciimax integer;
	asciimin integer;
	value varchar;
begin
	--选出最大值
	select max(x.a) into asciimax
	from
	(
        select distinct
        ascii(substr(t.title,n,1)) a
        substr(t.title,n,1)
        from (select fm as title) t
        cross join generate_series(1,length(t.title)) n
    )x;
    --选出最小值
    select min(x.a) into asciimin
    from
    (
        select distinct
        ascii(substr(t.title,n,1)) a,
        substr(t.title,n,1)
        from (select fm as title) t
        cross join
        generate_series(1,length(t.title)) n
    )x
    --有效字符
    where a > 127;
    
    --接下来用asciimax和asciimin判断区间就好
    if ((ascimax <= 740 and ascimax >= 0) or (ascimax >= 7424 and ascimax <=
8594) or
(ascimax >= 11360 and ascimax <= 11391) or (ascimax >= 42786 and ascimax
<= 43876))
then value = 'Latin';
elseif ((ascimin >= 880 and ascimin <= 1023) or (ascimin >= 7462 and ascimin
<= 8446))
then value = 'Greek';
elseif ((ascimin >= 1024 and ascimin <= 1327) or (ascimin >= 7296 and ascimin
<= 7544) or
(ascimin >= 42560 and ascimin <= 42655))
then value = 'Cyrillic';
elseif ((ascimin >= 1536 and ascimin <= 2303) or (ascimin >= 64336 and
ascimin <= 69246) or
(ascimin >= 124464 and ascimin <= 126705))
then value = 'Arabic';
elseif (ascimin >= 2304 and ascimin <= 3572)
then value = 'Indian';
elseif (ascimin >= 3585 and ascimin <= 3675)
then value = 'Thai';
elseif (ascimin >= 4096 and ascimin <= 4255)
then value = 'Burmese';
elseif ((ascimin >= 4352 and ascimin <= 4607) or (ascimin >= 12593 and
ascimin <= 12686) or
(ascimin >= 12800 and ascimin <= 12926) or (ascimin >= 43360 and
ascimin <= 55203) or
(ascimin >= 43360 and ascimin <= 55291) or (ascimin >= 65440 and
ascimin <= 65500))
then value = 'Korean';
elseif (ascimin >= 6016 and ascimin <= 6137)
then value = 'Khmer';
elseif ((ascimin >= 11904 and ascimin <= 12333) or (ascimin >= 12344 and
ascimin <= 12347) or
(ascimin >= 13312 and ascimin <= 42182))
then value = 'Chinese';
elseif ((ascimin >= 12353 and ascimin <= 12543) or (ascimin >= 12784 and
ascimin <= 12799) or
(ascimin >= 13008 and ascimin <= 13143))
then value = 'Japanese';
else value = 'Other';
end if;
return value;
end;
$$;
language plpgsql;
```

(3). **仅仅给定广东的车牌号，要求判断给定的车牌号是哪个市的？**

* 车牌号的规定：

a. 输入的车牌号的第一个字必须是`粤+字母`，如果第一个字不是`粤`,就要报异常`Invalid Province`.如果后面的字母不在广东省城市内，就报`Invalid city`

b. `粤+字母`以后，如果后面有五个字符，那这五个字符要么全是数字，要么全是大写字母,如果后面有6个字符，那么头一个字符必须使`D`或者`F`,剩下的要么是大写字母，要么是0-9的数字。

* 判断方法：

<font color=green>第一步：判断第一个字是不是“粤”。</font>

<font color=blue>第二步：获取字符串的长度</font>

<font color=red>第三步：判断字符串的长度在不在[7,8]范围内。</font>

<font color=orange>第四步：如果长度是8，判断第三个字符是不是‘D’或者‘F’</font>

<font color=purple>第五步：判断从第四个字母开始向后面取五个字符，这五个字符是不是要么全是大写字母，或者要么全是数字。****</font>可以用循环，也可以用正则表达式。

第六步：判断第二个字符代表的城市是不是广东省的城市。

c. 一个小技巧：在返回城市名称时，可以构造一个字符串列表，把这个字符串列表的下标和第二个字符的`ascii`码建立关系，之后用取字串的方法返回就可以了。

```sql
create or replace function get_city(car_num varchar)
returns varchar
as $$
declare
	--车牌字符串的长度
	cn_len integer;
	--车牌字符串第二个字符的ascii码
	city_number integer;
	--城市列表
	city_names varchar(80) := 'GUANG ZHOU,SHEN ZHEN,ZHU HAI,SHAN TOU,FO
SHAN,SHAO GUAN,ZHAN JIANG';

begin
	--检查粤
	if substr(car_num,1,1) <> '粤'
	then
		raise exception 'Invalid Province';
	end if;
	
	--赋值字符串长度
	cn_len = length(car_num);
	
	--判断字符串长度
	if not cn_len between 7 and 8
	then
		raise exception 'Invalid plate length';
	--判断长度为8的时候，第三个字符
	elseif cn_len = 8 and substr(car_num,3,1) not in ('F','D')
	then
		raise exception 'Invalid plate length';
	end if;
	
	--判断模式
	if not substr(car_num,cn_len-4,5) ~ '^[A-Z0-9]+$'
	then
		raise exception 'Invalid car number';
	end if;
	
	--获取第二个字符的ascii码
	city_number = ascii(substr(car_num,2,1));
	
	--判断是否在合理范围内
	if city_number not between 65 and 71 then
		raise exception 'Invalid city';
	--否则就符合题意
	else
		return split_part(city_names,',',city_number = 64);
	endif;
end;
$$
language plpgsql;
```





## Procedures（存储过程）



1. **概念：**存储过程**本质上就是把一系列的`SQL`命令封装成一个模块**，很像没有返回值的函数。
2. **使用`Procedures`的好处：**

（1）进行了模块化，简介。

（2）**优化传输：**数据库在云端，数据库的连接可能不是太快，如果我用存储过程，我就可以用一个很短的命令发到数据库中，**存储过程存储在云端的数据库**，这样就可以很快地调起来命令。

（3）**安全：**存储过程我们可以通过验证保证它的正确性，之后和数据库进行交互的话，如果要修改数据库中的内容，<font color=red>我可以规定，只有特定的存储过程可以修改，对于其他的修改命令，数据库有权拒绝。</font>

3. **实例：我要插入一部新的电影到`movies`表中**

（1）已知电影的这些属性：`title,country_name,year_released,director,actor1,actor2`

（2）首先，我要根据`country_name`，在`country`表里找到`country_code`，然后插入到`movies`表中

<font color=red>注意，弄完以后还要检查一下是否插入成功，比如一个国家的名字打错了，在`country`表中就找不到了</font>

```sql
insert into movies...
select country_code,..from countries
```

（3）我要根据`director`和`actor`的名字，在`people`表中找到`peopleid`，然后把这条记录添加到`credits`表中。

```sql
insert into credits...
select peopleid,'D',...
from people
```

**另一种思路：我先把导演或者演员的信息用`select`拼接一下，然后和`people`这个表按照条件`join`一下。**

<font color=red>还要检查是否插入成功</font>

```sql
insert into credits
select peopleid,...
from (
    select director,'D',...
    union all
    select actor1,'A',...
    union all
    select actor2,'A',...
)a
inner join people on...
```

（4）**利用Procedure实现**：

```sql
create function movie_registration
				(p_title varchar,
                 p_country_name varchar,
                 p_year	int,
                 p_director_fn varchar,
                 p_director_sn varchar,
                 p_actor1_fn varchar,
                 p_actor1_sn,varchar,
                 p_actor2_fn varchar,
                 p_actor2_sn varchar)
returns void
as $$
declare
	n_rowcount int;
	n_movieid int;
	n_people int;
	begin
		#插入电影
		insert into movies(title,country,year_released)
			select p_title,country_code,p_year
				from countries
					where country_name=p_country_name
		#检查插入是否成功
		#get diagnostics用于获取运行过程中的状态值
		get diagnostics n_rowcount = row_count;
		#如果没找到country,报异常
		if n_rowcount = 0
		then
			raise exception 'country not found in table COUNTRIES'
		end if;
		#获取刚插入的电影id
		#注意n_movieid声明在begin前面，赋值要用:=
		n_movieid := lastval()
		
		#获取真正要插入的人员的数量(防止NULL出现)
		select count(surname)
		into n_people
		from (select p_director_sn as surname
             	union all
              select p_actor1_sn as surname
             	union all
              select p_actor2_sn as surname) specified_people
              where surname is not null;
              
         #插入到credits
         insert into credits(movieid,peopleid,credited_as)
         #把给定的信息用select组成一个虚拟的表
         select (n_movieid,people.peopleid,provided.credited_as)
         from(
             select coalesce(p_director_fn,'*') as first_name,p_director_sn as surname,'D' as credited_as
             union all
             select coalesce(p_actor1_fn,'*') as first_name,p_actor1_sn as surname,'A' as credited_as
             union all
             select coalesce(p_actor2_fn,'*') as first_name,p_actor2_sn as surname,'A' as credited_as
         )provided
         #和people表inner join
         inner join people
         	on people.surname = provided.surname
         	and coalesce(people.first_name,'*') = provided.first_name
         #我给定的要插入的surname不能为NULL
         where provided.surname is not null;
        #检查是否插入成功
        get diagnostics n_rowcount = row_count;
        if n_rowcount != n_people
        	then
        	raise exception 'Some people couldn`t be found';
        endif;
	end;
   $$ language plpgsql;
```

调用过程：

```sql
select movie_registration('The Adventures of Robin Hood','United States',1938,'Michael','Curtiz')
```

4. **如果要从一个存储过程调用另一个存储过程，需要加上`perform`关键字。**

```sql
perform movie_registration(...)
```





## Trigger（触发）



1. **概念：**在操作数据的时候，可以**隐式**地触发一个存储过程的执行。
2. 只有在**修改数据**的时候才会触发。
3. **触发设计的目的：**

（1）比如别人要插入一条记录，我可以利用触发把它传进来的数据进行修改，然后再插入。比如插入`country`的时候，我规定要用小写，但是有的人就是插入了大写，我可以通过触发转换成小写。

<font color=red>问题1：在大批量插入数据的时候，我每次插入一条，都要用触发去判断，这样效率就不好。</font>

<font color=red>问题2：如果插入数据的用户用`select`和大写去查，没有查到怎么办？</font>

（2）**扩展了`constraints`:** 有些插入时的规范太过复杂，以至于用`constraints`无法实现，这个时候就可以在触发那里进行定义了。

（3）**存储电影的关键词：**比如我新插入了一部电影，我就可以把这部电影的`title`中的每个单词取出来，然后自动插入到存储关键词的表中

4. **触发激活的四种时间点：**根据**在这条命令之前，之后**和**在插入某一条记录之前，之后。**
5. **触发激活的时间：**`insert,update,delete`
6. **触发的定义格式：**

```sql
create trigger trigger_name
before insert or update or delete
on table_name
for each row#这里可以改成statement
when (条件) execute procedure 具体的存储过程的名称;
#别忘了;
```

7. **`instead of trigger`:**我一般的触发，都至少要做两件事，触发`procedure`和进行插入删除这些操作，但是，`instead of trigger`不会，==他只会执行`procedure`里面的程序。不管你的插入或者更新操作。==

8. **在每一行被添加删除修改之前之后的触发的定义方式：**

   **Before Trigger:**

```sql
create trigger trigger_name
	before insert or update or delete
	on table_name
	for each row
	when() execute procedure procedure_name(); 
```

​	**After Trigger:**

```sql
create trigger trigger_name
	after insert or update or delete
	on table_name
	for each row
	when() execute procedure procedure_name();
```

9. **实例：处理这样一个姓名字符串，把它变成中文名和英文名，同时处理国际生还是非国际生。**

也就是说，这个姓名字符串有三种格式：`“五远航, WU Yuanhang”`这是中国人的格式，国际生的格式有两种：`“Jonn C,John C”`,这一种，表明`name`和`ename`是一样的，是**西方人**直接插入就好，还有一种`“Juln,Lu”`这个表示只有`name`。需要自动生成`ename`

```sql
create or replace function new_design()
	returns trigger
as $$
declare
	#用于判断国际生是不是西方人
	n_count int;
	s_name students.name%type;
begin
	#new表示要插入的那一条记录
	s_name := new.name;
	#如果是中国人，先把字符串按照,分割，之后删除空白分别赋值
	if ascii(s_name) between 19968 and 40959
	then
		new.name := trim(split_part(s_name,',',1));
		new.ename := trim(split_part(s_name,',',2));
	else
		#把国际生的学号，原始字符串，逗号两边的字符串取出来
		with q as
		(
            select new.studentid,
            new.name,
            split_part(s_name,',',1) as part1,
            split_part(s_name,',',2) as part2
        )
        
        select count(*) into n_count
        from(
            #每个姓，名字符串之间按照空格进行分割
            select studentid,name,split_part(part1,' ',n) as part
            from q
            cross join generate_series(1,20) n
            except
            select studentid,name,split_part(part2,' ',n) as part
            from q
            cross join generate_series(1,20) n
        )x;
        #西方人
        if n_count = 0
        then
        	new.name := trim(split_part(s_name,',',1));
        	new.ename := trim(split_part(s_name,',',2));
        else
        	new.ename := trim(upper(trim(split_part(s_name,',',1)))) || ' '|| trim(split_part(s_name,',',2));
        end if;
      end if;
      return new;
return new;
end;
$$
language plpgsql;
```

**之后创建触发：**

```sql
create trigger students_trg
	before insert
	on students
	for each row
	when (new.ename is null)
	execute procedure new_design();
```

10.==提醒：用`with`代替表不能单独成一句话，需要和`select`查询连用。==

11. 

## Auditing（审计）



1. **概念：**如果有人修改了表中的记录，我就可以拿一个表去记录下来是谁在什么时候修改了这个记录。必要时需要查看。
2. **审计表**：对于`people`表的审计表

```sql
create table people_audit
(
    audit_id serial,
    peopleid int not null, #这个表里的哪个人被修改了？
    type_of_change char(1), #插入：I，删除：D
    column_name varchar, --哪个列被修改了
    old_value varchar,
    new_value varchar,
    changed_by varchar, --被谁修改了？
    time_changed datetime
);
```

另一种方式：把修改记录弄成`JSON`格式的字符串，然后存到磁盘上，必要时再放到数据库中。

3. **创建审计函数：**

（1）首先，这个函数**必须是在`trigger`中被调用的。**

（2）这个函数的参数列表必须为空。

（3）注意：修改时，<font color=red>列的修改情况是不一定的，可能只有一列被修改了，可能有很多列被修改了，所以我要检查每一列是否被修改了。</font>

```sql
create or replace function people_audit_fn()
returns trigger #表示这个函数是在trigger中被调用的
as $$
begin
	#tg_op是一个预定义的系统值，返回操作类型
	if tg_op = 'UPDATE'
	then
		insert into people_audit(peopleid,
                                 type_of_change,
                                 column_name,
                                 old_value,
                                 new_value,
                                 changed_by,
                                 time_changed)
        select peopleid,'U',column_name,old_value,new_value,
        		current_user || '@' || coalesce(cast(inet_client_addr() as varchar) ,'localhost'),current_timestamp
        #把所有被修改的列用union all弄成一个整体的表
        from (
            #检查first_name是不是被修改了
            select old.peopleid,
            		'first_name' as column_name,
            		old.first_name old_value,
            		new.first_name new_value,
            where coalesce(old.first_name,'*') <> coalesce(new.first_name,'*')
            union all
            #检查surname是不是被修改了
            select old.peopleid,
            	'surname' as column_name,
            	old.surname old_value,
            	new.surname new_value,
            where old.surname <> new.surname
            union all
            #检查born是不是被修改了
            select old.peopleid,
            	'born' as column_name,
            	cast(old.born as varchar) old_value,
            	case(new.born as varchar) new_value
            where old.born <> new.born
            #检查died是不是被修改了
            select old.peopleid,
            	'died' column_name,
            	cast(old.died as varchar) old_value,
            	case(new.died as varchar) new_value,
            where coalesce(old.died,-1) <> coalesce(new.died,-1)
            
        )modified
        
    #如果是insert操作,要用new来获取值
    elsif tg_op = 'INSERT' then
    	insert into people_audit(...)
    	...
    #如果是delete，要用old来获取值
    else
    	insert into people_audit(...)
    	...
    end if;
    #如果trigger时after each row的话，就return null就可以了
    #如果时before each row的话，就需要返回一些东西
    return null;
  end;
  $$ language plpgsql;
```

4. **创建审计触发:**

```sql
create triger people_trg
after insert or update or delete
on people
for each row
execute procedure people_audit_fn();
```

5. **`update`的混沌状态：**

（1）假设我现在有一个表定义如下：

```sql
create table test(
    id int,
    label varchar,
    unique(id)
);
```

（2）我向表中增加了两条数据：

```sql
insert into (id,label) values
						(1,'This is line 1'),
						(2,'This is line 2');
```

（3）现在，我有一个需求，要交换这两条记录的`id`，有下面两种写法

```sql
#不会报错的写法，命令执行时进入了混沌状态(Postgresql，sqlite会报错)
update test set id = case id when 1 then 2 else 1 end;

#会报错的写法,因为第一条命令执行完以后id就会重复
update test set id = 2 where id = 1;
update test set id = 1 where id = 2;
```

**如果没报错的话，我给这个表的`update`操作加一个`each ro trigger`，如果我更新了一行，这个表会触发`trigger`，此时会发生什么？**

5. **`Trigger`是最后一道防线：**当前面所有的措施保证不了数据的一致性和完整性的时候，`trigger`可以修正，但是==`trigger`很复杂，对性能损耗很大，所以尽量不要用`trigger`==，更不要用`trigger`去修复设计时的问题。





## Index（索引）



1. **数据库存储数据的具体方式：**首先，数据库会把数据存到一个个的文件`file`中，之后，数据库会把一个文件分成几个`block`，对于每个`block`,数据库会存储一下从`block`开头多少个字节的偏移量是一条记录的开始。

2. **数据库检索数据的方式：**首先，数据库根据`value`，去找到`locator`，这个`locator`知道以后，我就知道了数据所在的具体的位置，之后去这个位置匹配值就可以了。注意这个`value`是一个向量，它可以是多个`value`的组合。找`locator`的过程利用了**`B`树**。
3. **创建索引的方式：**

```sql
create index 索引名称
on table_name (col1,col2,...);
```

比如：对于`countries`表的`continent`建立索引。

```sql
create index countries_cont_idx
on countries(continent);
```

或者：`unique`可以保证这几个列的组合不重复。

```sql
create unique index people_surname_born_idx
on people(surname,born);
```

4. **自动生成索引：**如果一列是`primary key`或者有`unique constraint`，那么这一列会自动生成索引。
5. **索引的本质：**为这个列创建**B树形结构**。
6. **需不需要把所有的列创建索引？**

（1）如果创建了索引，在`insert`或者`delete`之后，这个表的索引可能要重新维护，插入一条数据可能就会修改无数索引，这样操作==效率会非常低。==

（2）如果创建了索引，拿数据库就需要空间存这颗B树，如果创建太多索引，那么就可能导致==索引占的空间就会比数据占的空间多==。

7. **尽量用`unique constraint`**：索引代价有点大。
8. **索引的命名规范：**`tablename_col1_col2_..._idx`



## Performance（性能）



1. **分析性能的消息队列模型：**

（1）在数据库中，查询的请求会排成队列等待数据库进行处理。

（2）**如果数据库处理的速度小于队列堆积的速度，那么这个查询就会越来越慢。**

2. **索引不一定能够加速：**

（1）如果我现在有一个需求，需要按照一个列表`list`去查询对应的数据。

（2）利用全表扫描的方式，时间复杂度为$O(n)$,但是利用索引，复杂度就是$O(nlogn)$，这种情况下索引就慢了。

<font color=red>所以，当返回的数据多的时候，用全表扫描比较好，当返回的数据少的时候，用索引比较好。数据库会自动选择。</font>

3. **使用索引的选择：**<font color=blue>查询快和修改快两者不可兼得</font>

![索引比较](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\索引比较.png)

4. **应该为哪些列创建索引？**

   ==看平时检索的时候哪些列用的最频繁，就为哪些列创建索引。==

   ==这个列要具有区分性，就是和每一个值对应的记录条数要尽量少。==

   比如书名，加索引，这个索引就不好。或者电影表中用国家名，这也不好。

5. **索引不会被使用的情况：**==本质上就是无法排序==

（1）如果这个index是由几个列组成的，你在`where`条件里面只使用了其中的几个，那么索引就不会被使用，因为破坏了排序的规则。

（2）如果查询条件中有`like '%something%'`**（前面还有东西）**,或者有了`substr(2,..)`**（第一个字符没包含)**，那么索引也不会被使用。

（3）如果我给列`c`建立了索引，但是查询条件中用了函数`f(c) = ...`，就有可能破坏排序规则，就可能不用索引了。

（4）设想如下场景：我在检索条件中写了`where upper(surname) = 'MARVIN'`，之后我数据的排序结果如下：其中`Stewart`是根，那么我在检索`marvin`的时候，遍历的时候就不知道是向左还是向右了，这种情况下索引不会使用。

```sql
MARVIN
MILES
O'brien
Stewart
marvin
wayne
```

（5）如果我为`data_time`建立了索引，但是我在`where`中写的时候，我把不同的部分（年、月、日）分开了，那这种情况下就破坏了索引，解决方法是：==我用一个区间去搜索`data_time`。==

```sql
#错误写法
where extract(month from data_column) = 6
and extract(year from data_column) = extract(year from current_date)
```

（6）**隐式类型转换问题：**如果我在`where`条件中写了`varchar_code=12345678`,**数据库底层会把这两个值都转换为浮点数，然后再进行比较，当我把所有`varchar_code`都转换成浮点数的时候，就会破坏原来的排序规则，于是索引就不再生效了。**

（7）**非确定性函数不能建立索引。**

==为了使索引生效，最好在插入数据的时候，就要保证数据的规范性。==



## Execution Plan（执行计划）



1. **怎么知道一条命令的执行细节？–执行计划**

```sql
explain 你想知道的sql命令
```

2. **注意：**不同的数据库管理系统，对于相同的命令，执行计划是不同的。
3. **通过执行计划，可以看到索引是否被使用。**





## View（视图）

1. **概念：**视图相当于一个虚拟的表，这个表通过`select`命令选择出来，封装，然后链接到真实的表中，**当真实的表数据发生变化时，视图也会同步改变。****是一种对数据库关系操作的重用。**
2. **定义格式：**`create view 视图的名称 as 后面的select语句`
3. **实例：把国家的全名和电影的名称链接起来**

```sql
create view vmovies as 
select m.movieid,m.title,m.year_released,c.country_name
from movies m inner join countries c
on m.country = c.country_code
```

使用方法：直接当成表来用

```sql
select *
from vmovies
where country_name = 'Italy'
```

4.==视图的存在可以优化范式带来的用户不友好的体验！==

5. **视图可以看作从不同视角下观察数据的一种方式**，就好像图像在不同的线性空间下进行观察的结果。
6. 聪明的人设计视图，愚蠢的人使用视图。
7. **实例：电影和演员，导演信息的对应视图：**

```sql
create view vmovie_credits
as select m.title,
		  m.year_released release,
		case c.credited_as
			when 'A' then 'Actor'
			when 'D' then 'Director'
			else '?'
			end duty,
			full_name(p.first_name,p.surname) name
	from movies m
		inner join credits c
			on c.movieid = m.movieid
		inner join people p
			on p.peopleid = c.peopleid
```

8. **对于上述例子中，`name`这一列是由函数定义的，由于`view`不是实体数据，如果我运行如下命令：**

```sql
select * 
from vmovie_credits
where name = 'Humphrey Bogart'
```

**数据库会把原表中所有的数据先调用这个函数，然后再进行对比，这就特别花时间。**

9. `view`上面可以继续建立`view`，但是性能会进一步损失。
10. **一般情况下，不可以根据视图去修改数据**，我的视图可能由很多表`join`得到，可能是聚合函数的结果，可能是函数的返回值，也可能修改了这个数据需要在**原表中有但是视图中没有的列上添加信息。**
11. **视图可以修改原数据的一个例子：**

（1）假设我有一个表`user_scope`,有两列，叫`username,continent`，我现在有一个需求，只能让登录的这个用户看到他所管的大洲的数据。

```sql
create or replace view vmy_movies
as select m.movieid,
		  m.title,
		  m.year_released,
		  m.country,
	from movies m
	where m.country in
	(select c.country_code
    	from countries c
    		inner join user_scope u
    			on u.continent = c.continent
    	where u.username = user)
```

（2）对于上面这个列表，每个用户登录进行查询的结果都不相同。

（3）**问题：**注意，上面这个表中，`country_code`和`continent`是在两个表中存放的，假设我现在视图中修改，把一个亚洲的`country_code`修改成了欧洲的`country_code`,那么和`country`表进行`inner join`的时候，这条记录就会消失，如果我修改错了，那么我再也没有办法去找回修改错误的数据了。还比如如果我是亚洲的管理员，我通过视图插入了一条美国电影，我可以插入，但是我**永远检索不到**。

（4）上述问题的解决方法：在创建`view`的时候，在后面加上一句话叫==`with check option`==,这样当你`update`或者`delete`的时候，数据库就会检查你修改或者插入后，数据会不会还在你这个视图。

（5）上述问题的破解方法：**利用`instead of trigger`，表面上用`insert`语句触发，然后利用存储过程进行修改，这样就可以绕过`with check option`的限制。**





## Security（安全性）

1. **数据库的访问需要授权：**最常见的实现方式就是通过`username`和`password`。
2. **数据库的对不同用户的权限分配：**`grant`关键词把某一个表的某种操作的权限分配给用户。

```sql
grant select on tablename to accountname
grant insert on tablename to accountname
grant update on tablename to accountname
grant delete on tablename to accountname
grant select,insert on tablename to accountname
```

3. **权限收回：**`revoke`关键词

```sql
revoke select on tablename from accountname
```

4. **利用视图保证数据的安全性：**

（1）比如，我有一个表，我只希望用户看到数据的前三列，后面两列不希望人看到。

（2）我可以用一个`view`把前三列包装起来。

（3）之后用`grant select on viewname to accountname`来把权限给到`view`上面。

5. **利用视图保护某些行的数据：**比如我的员工管理系统，我只能让员工自己看到自己的数据。

```sql
create view my_stuff
as
select * from stuff
where username = user
```

**注意：这个`user`指的是用户当前登录数据库使用的这个`username`,而`username`这个列属性表示：这条数据只能给哪个人看。**

5. **`create or replace view`的用法：**

（1）比如我现在有一个`view`，我给很多用户都分配了权限，现在我发现这个`view`有缺陷，我要重新修改一下，如果先删除再修改，**权限也就消失了。**

（2）但是，如果我用`create or replace view`，我就能在**不取消权限分配的基础上，增加我要修改的东西。**





## Data Dictionary（数据字典）



1. **概念：**数据字典又叫`Catalog`，为我们提供了所有数据库的描述信息。
2. **Schemas:**有点像视图，这就是一个**虚拟的数据库**，随时随地链接到我真实的数据库。注意：==`foreign key`不能跨数据库指向，但是可以跨Schema指向。==

3. **DDL操作，本质上是通过修改<font color=red>系统表</font>来实现的，这些系统表是通过`system view`来进行读取信息的。**这些`system view`在`information_schema`中,这个schema有很多view。

**查询这个数据库中所有表和视图的信息**

```sql
select * from information_schema.tables
where table_schema = 'public';
```

![数据字典](C:\Users\李家奥\Desktop\计算机知识体系\数据库原理和SQL语言\Picture\数据字典.png)

**查询数据库中某个表的列的信息**

```sql
select * from information_schema.columns
where table_name = 'movies'
```

4. **可以利用数据字典高效对表进行删除：**

**把数据库中所有的以`TMP`开头的表删除：**

```sql
select 'drop table ' || table_name || ';'
from information_schema.tables
where table_name like 'TMP%'
and...
```

5. 数据字典里面的信息叫做`meta-data`



## NoSQL

1. `no sql` = `not only sql`
2. 有很多数据库都不是关系型的。



## Query Optimize（查询优化）

1. **数据库查询的具体过程：**

（1）客户端会发命令到HTTP服务器的端口。

（2）之后，HTTP服务器会把命令包装成sql命令发送到数据库服务器的端口。

（3）之后，数据库服务器会把结果返回，然后HTTP服务器进行包装，以网页的形式显示到客户端。

2. **查询优化的方法：**

（1）把数据字典里面的信息`meta-data`放到缓存里面去，这样，对于这个表多次查询的读，写就快了。

（2）语法分析`Parse`是非常花时间的，我们可以把已经分析好的sql语句放到内存中，这样重复周期性地调用某些sql语句的时候，就可以加速。

（3）`LRU(Least Recently Used)`:根据命令使用的频繁程度，动态管理缓存，节省`IO`成本。



## Scaling（扩容）



1. **不要把数据放到一台服务器上，可以把数据放到多台服务器上，能保证性能和安全性。**
2. **可以通过高速的内部网，把不同的服务器进行连接，实现多台数据库进行连接。**
3. **服务器的总体性能和服务器的数量不成正比：**因为服务器之间的通信需要代价。（Universal Law of Computational Scalability)
4. **事务的实现方式：**在多台服务器之间，分别进行`commit`，如果有一台服务器不行了，那就失败。

   