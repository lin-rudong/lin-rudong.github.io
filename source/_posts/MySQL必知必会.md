---
title: MySQL必知必会
date: 2019-03-20 21:29:06
tags:
categories: MySQL 
---

# 使用MySQL

## 选择数据库

最初连接到MySQL时,没有任何数据库打开供你使用,在执行任意数据库操作前,需要选择一个数据库.

```mysql
use d
```

## 了解数据库和表

```mysql
show databases
show tables

-- 等价
show columns from t
describe t

show create database d
show create table t
show grants

show status -- 服务器状态信息
show errors
show warnings
```

# 检索数据

## 检索所有列

一般,除非你确实需要表中的每个列,否则最好别使用*通配符,因为检索不需要的列通常会降低检索和应用程序的性能.

## 检索不同的行

```mysql
select distinct i from t
```

不能部分使用distinct,distinct关键字应用于所有列.

## 限制结果

```mysql
select i from t limit 5 -- 0,1,2,3,4
select i from t limit 5,5 -- 5,6,7,8,9
select i from t limit 4 offset 3 -- limit 3,4
```

# 排序检索数据

## 排序数据

```mysql
select i from t order by i
```

order by子句中使用的列可以是非检索的列.

## 按多个列排序

```mysql
select i,n from t order by i,n
```

## 指定排序方向

默认是升序排序,为了进行降序排序,必须指定desc关键字.

```mysql
select i from t order by i desc
```

desc关键字只应用到直接位于其前面的列明,如果想在多个列上进行降序排序,必须对每个列指定desc关键字.

与desc相反的关键字是asc.

order by子句的位置.

```mysql
select i from t order by i limit 5
```

# 过滤数据

## 使用where子句

where子句的位置.

```mysql
select i from t where i=1 order by i
```

## where子句操作符

MySQL支持的条件操作符有=,<>,!=,<,<=,>,>=,between.

### 检查单个值

MySQL在执行匹配时默认不区分大小写.

### 范围值检查

```mysql
select i from t where i between 5 and 10
```

between匹配范围中所有的值,包括指定的开始值和结束值.

### 空值检查

```mysql
select i from t where i is null
```

# 数据过滤

## 组合where子句

### and操作符

### or操作符

### 计算次序

任何时候使用具有and和or操作符的where子句,都应该使用圆括号明确地分组操作符.

## in操作符

in操作符优点具体如下.

1. in操作符一般比or操作符清单执行更快.
2. in的最大优点是可以包含其他select语句,使得能够更动态地建立where子句.

功能与or相当.

## not操作符

# 用通配符进行过滤

## like操作符

### 百分号(%)通配符

%表时任何字符出现任意次数.

```mysql
select i from t where i like 'zi%'
```

%除了一个或多个字符外,还能匹配0个字符.

%不能匹配null.

### 下划线(_)通配符

_只匹配单个字符,不能多也不能少.

## 使用通配符的技巧

除非绝对有必要,否则不要把它们用在搜索模式的开始处,这样搜索起来是最慢的.

# 用正则表达式进行搜索

## 使用MySQL正则表达式

### 基本字符匹配

`.`表示匹配任意一个字符.

regexp关键字,其与like有一个重要的差别,like匹配整个列,而regexp在列值内进行匹配,如果regexp要和like一样,需要使用^和$定位符.

匹配不区分大小写,为区分大小写,可使用regexp binary.

### 进行or匹配

`|`为正则表达式的or操作.

### 匹配几个字符之一

`[]`匹配任何指定的单一字符,[]是另一种形式的or语句.

```mysql
select i from t where i regexp '[123]'
```

字符集合也可以被否定,在集合的开始处防止一个^即可.

### 匹配范围

集合可使用`-`来定义一个范围.

### 匹配特殊字符

为了匹配特殊字符,必须用`\\`转义.

空白元字符包括,`\\f`换页,`\\n`换行,`\\r`回车,`\\t`制表,`\\v`纵向制表.

为了匹配反斜杠字符本身,需要使用`\\\`.

MySQL要求两个反斜杠,MySQL自己解释一个,正则表达式库解释另一个.

### 匹配多个实例

重复元字符.

1. `*`匹配0个或多个匹配,等于{0,}.
2. `+`1个或多个匹配,等于{1,}.
3. `?`0个或1个匹配,等于{0,1}.
4. `{n}`指定数目的匹配.
5. `{n,}`不少于指定数目的匹配.
6. `{n,m}`匹配数目的范围,m不超过255.

### 定位符

1. `^`文本的开始.
2. `$`文本的结尾.
3. `[[:<:]]`词的开始.
4. `[[:>:]]`词的结尾.

简单的正则表达式测试,0表示没有匹配,1表示匹配.

```mysql
select 'hello' regexp 'h.'
```

# 创建计算字段

## 计算字段

计算字段是运行时在select语句中创建的.

## 拼接字段

多数DBMS使用+或||来实现拼接,MySQL则使用`concat()`函数来实现.

别名用`as`关键字赋予,别名有时也称为导出列.

## 执行算数计算

支持的基本算数操作符,`+`,`-`,`*`,`/`.

select提供了测试和试验函数与计算的一个很好的办法.

# 使用数据处理函数

## 使用函数

1. 文本函数.
2. 数值函数.
3. 日期和时间函数.
4. 系统函数.

### 文本处理函数

1. `left()`返回串左边的字符.
2. `right()`返回串右边的字符.
3. `length()`返回串的长度.
4. `locate()`找出串的一个字串.
5. `substring()`返回字串的字符.
6. `soundex()`返回串的soundex值,soundex是一个和发音相关的值.
7. `lower()`将串转换为小写.
8. `upper()`将串转换为大写.
9. `ltrim()`去掉串左边的空格.
10. `rtrim()`去掉串右边的空格.
11. `trim()`去点串两边的空格.

### 日期和时间处理函数

1. `now()`返回当前日期和时间.
2. `curdate()`返回当前日期.
3. `curtime()`返回当前时间.
4. `date()`返回日期时间的日期部分.
5. `time()`返回日期时间的时间部分.
6. `day()`返回日期的天数部分.
7. `month()`返回日期的月份部分.
8. `year()`返回日期的年份部分.
9. `dayofweek()`返回日期对应的星期.
10. `hour()`返回时间的小时部分.
11. `minute()`返回时间的分钟部分.
12. `second()`返回时间的秒部分.
13. `adddate()`增加一个日期,date_add()同义.
14. `addtime()`增加一个时间.
15. `datediff()`计算两个日期之差.
16. `date_format`返回一个格式化的日期或时间串.

### 数值处理函数

1. `abs()`绝对值.
2. `mod()`除操作的余数.
3. `rand()`随机数.
4. `sqrt()`开方根.
5. `pow()`指数值.
6. `pi()`圆周率.
7. `exp()`e的指数值.
8. `cos()`余弦.
9. `sin()`正弦.
10. `tan()`正切.

# 汇总数据

## 聚集函数

聚集函数运行在行组上,计算和返回单个值的函数.

1. `avg()`列的平均值.
2. `count()`列的行数.
3. `max()`列的最大值.
4. `min()`列的最小值.
5. `sum()`列值之和.

都会忽略值为null的行,除了count(*)的用法.

## 聚集不同值

```mysql
select avg(distinct i) from t
```

distinct必须使用列名,不能用于count(*),用于min()和max()没有意义.

# 分组数据

## 创建分组

1. group by子句可以包含任意数目的列,为数据分组提供更细致的控制.
2. 所有null值将作为一个分组返回.
3. group by子句必须出现在where子句之后,order by子句之前.

使用with rollup关键字,可以得到每个分组以及每个分组汇总级别的值.

## 过滤分组

where过滤行,having过滤分组.

where在数据分组前进行过滤,having在数据分组后进行过滤,where排除的行不包括在分组中.

## select子句顺序

1. select.
2. from.
3. where.
4. group.
5. having.
6. order by.
7. limit.

# 使用子查询

## 利用子查询进行过滤

子查询一般与in操作符结合使用,但也还可以和条件操作符结合.

## 作为计算字段使用子查询

# 联结表

## 创建联结

### where子句的重要性

select语句中联结几个表时,相应的关系是在运行中构造的,在数据库表的定义中不存在能指示MySQL如何联结的东西.

```mysql
select t.i as ti,u.i as ui 
from t,u
where...
```

在from子句中联结的结果是笛卡尔积,检索出的行的数目将是第一个表中的行数乘以第二个表中的行数.

### 内部联结

from子句的联结默认是内部联结,也称为等值联结.

用inner join和on关键字比较规范,传递给on的实际条件与传递给where的相同,都是过滤作用.

```mysql
select t.i as ti,u.i as ui 
from t inner join u
on...
```

# 创建高级联结

## 使用不同类型的联结

### 自联结

用自联结而不用子查询,虽然最终的结果是相同的,但有时候处理联结远比处理子查询快得多.

### 外部联结

联结包含了那些在相关表中没有关联行的行,这种类型的联结称为外部联结.

```mysql
select t.i as ti,u.i as ui
from t left outer join u
on t.i=u.i
```

必须使用right或left关键字指定包括其所有行的表.

# 组合查询

## 组合查询

有两种基本情况,需要使用组合查询.

1. 在单个查询中从不同的表返回类似结构的数据.
2. 对单个表执行多个查询,按单个查询返回数据.

组合相同表的两个查询与具有多个where子句条件的单条查询完成的工作相同,这两种技术在不同的查询中性能也不同.

## 创建组合查询

可用union操作符来组合数条查询,将它们的结果组合成单个结果集.

### union规则

1. union中的每个查询必须包含相同的列,表达式或聚集函数,不过各个列不需要以相同的次序列出.
2. 列数据类型必须兼容.

### 包含或取消重复的行

union从查询结果集中自动去除了重复的行,换句话说,它的行为与单条select语句中使用多个where子句条件一样,这是union的默认行为,如果想要返回所有匹配的行,可使用union all.

### 对组合查询结果排序

在用union组合查询时,只能使用一条order by子句,它必须出现在最后一条select语句之后.

# 全文本搜索

通配符和正则表达式的几个重要的限制.

1. 性能,通配符和正则表达式匹配通常要求MySQL尝试匹配表中所有行,而且这些搜索极少使用表索引,因此,由于被搜索行数不断增加,这些搜索可能非常耗时.
2. 明确控制,很难明确地控制匹配什么和不匹配什么.
3. 智能化的结果,虽然这些搜素提供了非常灵活的搜索,但它们都不能提供一种智能化的选择结果的方法,例如按照可能是更好的匹配来排列它们,类似,一个特殊词的搜索将不会找出不包含该词但包含其他相关词的行.

在使用全文本搜索时,MySQL不需要分别查看每个行,不需要分别分析和处理每个词,MySQL创建指定列中各词的一个索引,搜索可以针对这些词进行,这样,MySQL可以快速有效地决定哪些词匹配,哪些词不匹配,它们匹配的频率等等.

## 使用全文本搜索

为了进行全文本搜索,必须索引被搜索的列,在索引之后,select可与match()和against()一起使用以实际执行搜索.

### 启用全文本搜索支持

```mysql
create table t(
	id int not null auto_increment,
	t text null,
	primary key(id),
	fulltext(t)
)engine=MyISAM
```

在定义之后,MySQL自动维护该索引,在增加,更新或删除行时,索引随之自动更新.

不要在导入数据时使用fulltext,更新索引要花时间,应该首先导入所有数据,然后再修改表,定义fulltext.

### 进行全文本搜索

在索引之后,使用两个函数match()和against()执行全文本搜索,其中match()指定被搜索的列,against()指定要使用的搜索表达式.

```mysql
select t from t
where match(t) against('lin')
```

除非使用binary方式,否则全文本搜索不区分大小写.

全文本搜索的一个重要部分就是对结果排序,具有较高等级的行先返回.

全文本搜索提供了简单like搜索不能提供的功能,而且由于数据是索引的,全文本能搜索还相当快.

### 使用查询扩展

利用查询扩展,能找出可能相关的记过,即使它们并不精确包含所查找的词.

```mysql
select t from t
where match(t) against('lin' with query expansion)
```

MySQL对数据和索引进行两边扫描来完成搜索.

1. 首先,进行一个基本的全文本搜索,找出与搜索条件匹配的所有行.
2. 其次,MySQL检查这些匹配行并选择所有有用的词.
3. 再其次,MySQL再次进行全文本搜索,这次不仅使用原来的条件,而且还使用所有有用的词.

### 布尔文本搜索

1. 要匹配的词.
2. 要排斥的词.
3. 排列提示,指定某些词比其他词更重要.
4. 表达式分组.
5. 另外一些内容.

布尔方式即使没有fulltext索引也可以使用,但这时一种非常缓慢的操作,其性能将随着数据量的增加而降低.

```mysql
select t from t
where match(t) against('lin -hello*' in boolean mode)
```

`-hello*`排除以hello开始的词.

全文本布尔操作符.

1. `+`包含,词必须存在.
2. `-`排除,词必须不出现.
3. `>`包含,而且增加等级值.
4. `<`包含,且减少等级值.
5. `()`把此组成子表达式,允许这些子表达式作为一个组被包含,排除,排列等.
6. `~`取消一个词的排序值.
7. `*`词尾的通配符.
8. `""`定义一个短语,与单个词的列表不一样,它匹配整个短语以便包含或排除这个短语.

### 全文本搜索的使用说明

1. 短词被忽略且从索引中排除,短语默认为3个或3个以下字符的词.
2. MySQL带有一个内建的非用词列表.
3. 许多次出现的频率很高,搜索它们没有用处,返回太多的结果,因此,MySQL规定了一条50%规则,如果一个词出现在50%以上的行中,则将它作为一个非用词忽略,50%规则不用于in boolean mode.
4. 如果表中的行数少于3行,则全文本搜索不返回结果,因为每个词或者不出现,或者至少出现在50%的行中.
5. 忽略词中的单引号.
6. 不具有词分歌词的语言不能恰当地返回全文本搜索结果.
7. 仅在MyISAM数据库引擎中支持全文本搜索.

# 插入数据

## 插入完整的行

```mysql
insert into t(i) values(1)
```

省略列,该列定义为默认值,或者null值.

```mysql
insert low_priority into
```

降低优先级.

## 插入多个行

```mysql
insert into t(i) values(1),(2)
```

可以提高数据库处理的性能,因为MySQL用单条insert语句处理多个插入比使用多条insert语句快.

## 插入检索出的数据

```mysql
insert into t(i) select i from t
```

# 更新和删除数据

## 更新数据

```mysql
update t set i=i+1,n=n+1 where i=1
```

在update语句中可以使用子查询,使得能用select语句检索出的数据更新列数据.

如果用update语句更新多行,并且在更新这些行中的一行或多行时出现一个错误,则整个update操作被取消,为即使是发生错误,也继续进行更新,可使用ignore关键字.

```mysql
update ignore t...
```

如果允许null值,删除某个列的值,可设置它为null.

## 删除数据

```mysql
delete from t where i=1
```

delete删除表的内容而不是表.

如果想从表中删除所有行,不要使用delete,可使用truncate table语句,它完成相同的工作,但速度更快,truncate实际是删除原来的表并重新创建一个表,而不是逐行删除表中的数据.

# 创建和操纵表

## 创建表

### 表创建基础

```mysql
create table t(
	i int not null auto_increment,
    n int not null default 1,
	primary key(i)
)engine=InnoDB
```

如果仅想在一个表不存在时创建它,应该在表名后给出if not exists,它只是查看表名是否存在,并且仅在表名不存在时创建它.

### 主键再介绍

主键必须唯一非空.

### 使用auto_increment

每个表只允许一个auto_increment列,而且它必须被索引,通常使它成为主键.

用last_insert_id()函数返回最后一个auto_increment值.

### 指定默认值

MySQL不允许使用函数作为默认值,它只支持常量.

使用默认值而不是null值,特别是对用于计算或数据分组的列更是如此.

### 引擎类型

1. InnoDB是一个可靠的事务处理引擎,它不支持全文本搜索.
2. MEMORY在功能等同于MyISAM,但由于数据存储在内存而不是磁盘,速度很快,特别适合于临时表.
3. MyISAM是一个性能极高的引擎,它支持全文本搜索,但不支持事务处理.

引擎类型可以混用,不同的表可以用不同的引擎.

外键不能跨引擎.

## 更新表

```mysql
alter table t add n int
alter table t drop column n

alter table t 
add constraint fk 
foreign key(fi) references f(i)
```

## 删除表

```mysql
drop table t
```

## 重命名表

```mysql
rename table t to t1
```

# 使用视图

## 视图

### 为什么使用视图

1. 重用SQL语句.
2. 简化复杂的SQL操作,在编写查询后,可以方便地重用它而不必知道它的基本查询细节.
3. 使用表的组成部分而不是整个表.
4. 保护数据.
5. 更改数据格式和表示.

因为视图不包含数据,所以每次使用视图时,都必须处理查询执行时所需的任一个检索,如果你用多个联结和过滤创建了复杂的视图或者嵌套了视图,可能会发现性能下降得很厉害,因此,在部署使用了大量视图的应用前,应该进行测试.

### 视图的规则和限制

1. 与表一样,视图必须唯一命名,不能给视图取与别的视图或表相同的名字.
2. 对于可以创建的视图数目没有限制.
3. 为了创建视图,必须具有足够的访问权限.
4. 视图可以嵌套,即可以利用从其他视图中检索数据的查询来构造一个视图.
5. order by可以用在视图中,但如果从该视图检索数据select中也含有order by,那么该视图中的order by将被覆盖.
6. 视图不能索引,也不能有关联的触发器或默认值.
7. 视图可以和表一起使用.

## 使用视图

```mysql
create view v
show create view v
drop view v
create or replace view v
```

### 利用视图简化复杂的联结

创建可重用的视图.

```mysql
create view v as
select i from t
where i in (1,2,3)
```

### 更新视图

视图是可更新的,更新一个视图将更新其基表.

并非所有视图都是可更新的,基本上可以说,如果MySQL不能正确地确定被更新的基数据,则不允许更新,如果视图定义中有以下操作,则不能进行视图的更新.

1. 分组.
2. 联结.
3. 子查询.
4. 并.
5. 聚集函数.
6. distinct.
7. 导出计算列.

视图主要用于数据检索.

# 使用存储过程

## 存储过程

存储过程简单来说,就是为以后的使用而保存的一条或多条MySQL语句的集合.

## 为什么要使用存储过程

1. 通过把处理封装在容易使用的单元中,简化复杂的操作.
2. 由于不要求反复建立一系列处理步骤,这保证了数据的完整性,如果所有开发人员和应用程序都是用统一存储过程,则所使用的代码都是相同的,这一点的延伸就是防止错误,需要执行的步骤越多,出错的可能性就越大,防止错误保证了数据的一致性.
3. 简化对变动的管理,如果表名,列名或业务逻辑有变化,只需要更改存储过程的代码,使用它的人员甚至不需要知道这些变化,这一点的延伸就是安全性,通过存储过程限制对基础数据的访问减少了数据讹误的机会.
4. 提高性能,因为使用存储过程比使用单独的SQL语句要快.
5. 存在一些只能用在单个请求中的MySQL元素和特性,存储过程可以使用它们来编写功能更强更灵活的代码.

换句话说,使用存储过程有3个主要的好处,即简单,安全,高性能,不过,存储过程也有它的一些缺陷.

1. 一般来说,存储过程的编写比基本SQL语句复杂,编写存储过程需要更高的技能,更丰富的经验.
2. 你可能没有创建存储过程的安全访问权限.

## 使用存储过程

### 执行存储过程

```mysql
call p()
```

### 创建存储过程

```mysql
create procedure p()
begin
	select i from t
end
```

### 删除存储过程

存储过程在创建之后,被保存在服务器上以供使用,直至被删除.

```mysql
drop procedure p
drop procedure if exists p
```

### 使用参数

```mysql
create procedure p(
    out a decimal(8,2)
)
begin
	select min(i) into a from t
end
```

关键字out指出相应的参数用来从存储过程传出一个值,返回给调用者,MySQL支持in,传递给存储过程,out从存储过程传出和inout对存储过程传入和传出,检索值用into保存到相应的变量.

```mysql
call p(@result)

select @result
```

所有MySQL变量都必须以@开始.

select可以显示结果.

### 建立智能存储过程

```mysql
create procedure p(
	in c boolean
)comment 'this is a procedure'
begin
	declare a decimal(8,2)
	declare b int default 1

	if c then
		...
	end if
end
```

declare语句定义局部变量,要求指定变量名和数据类型,它也支持可选的默认值.

comment值可以用show procedure status显示.

if语句还支持elseif和else子句,elseif还是用then子句,else不使用.

### 检查存储过程

```mysql
show create procedure p
show procedure status like 'f'
```

# 使用游标

## 游标

MySQL检索操作返回一组称为结果集的行,有时,需要在检索出来的行中前进或后退一行或多汗给,这就是是使用游标的原因,游标是语句检索出来的结果集,应用程序可以根据需要滚动或浏览其中的数据.

游标主要用于交互式应用,其中用户需要滚动屏幕上的数据,并对数据进行浏览或做出更改.

不像多数DBMS,MySQL游标只能用于存储过程.

## 使用游标

1. 在能够使用游标前,必须声明定义它,这个过程实际上没有检索数据,只是定义要使用的select语句.
2. 一旦声明后,必须打开游标以供使用,这个过程用前面定义的select语句把数据实际检索出来.
3. 对于填有数据的游标,根据需要取出检索各行.
4. 在结束游标使用时,必须关闭游标.

### 创建游标

```mysql
create procedure p()
begin
	declare c cursor
	for
	select i from t
end
```

### 打开和关闭游标

```mysql
open c
close c
```

游标关闭后,如果没有重新打开,则不能使用它,但是生命过的游标不需要再次声明,用open语句打开它就可以了.

如果你不明确关闭游标,MySQL将会在到达end语句时自动关闭它.

### 使用游标数据

```mysql
create procedure p()
begin
	declare o int
	declare done boolean default 0

	declare c cursor
	for
	select i from t
	
	declare continue handler
	for sqlstate '02000' 
	set done=1
	
	open c
	
	repeat 
		fetch c into o
	until done end repeat
	
	close c
end
```

continue handler在条件出现时被执行,sqlstate '02000'是一个未找到条件,当游标没有更多的行时出现这个条件.

除repeat语句外,MySQL还支持循环语句,直到使用leave语句手动退出位置,通常repeat语句的语法使它更适合于对游标进行循环.

# 使用触发器

## 触发器

触发器是MySQL响应delete,insert和update语句而自动执行的一条MySQL语句,或位于begin和end语句之间的一组语句.

## 创建触发器

1. 唯一的触发器名.
2. 触发器关联的表.
3. 触发器应该响应的活动.
4. 触发器何时执行,处理之前或之后.

```mysql
create trigger tr after insert on t
for each row
insert into u values('added')
```

for each row指定了对每个插入行执行一次.

只有表才支持触发器,视图不支持,临时表也不支持.

每个表最多支持6个触发器,每条insert,update,delete的之前和之后,单一触发器不能与多个时间或多个表关联.

如果before触发器失败,则MySQL将不执行请求的操作,此外,如果before触发器或语句本身失败,MySQL将不执行after触发器.

## 删除触发器

```mysql
drop trigger tr
```

触发器不能更新或覆盖.

## 使用触发器

### insert触发器

1. insert触发器代码内,可引用一个名位new的虚拟表,访问被插入的行.
2. 在before insert触发器中,new中的值也可以本更新,允许更改被插入的值.
3. 对于auto_increment列,new在insert执行之前包含0,在insert执行之后包含新的自动生成值.

```mysql
create trigger tr after insert on t
for each row
insert into u values(new.i)
```

通常,将before用于数据验证和净化,也适用于update触发器.

### delete触发器

1. 在delete触发器代码内,可以引用一个名为old的虚拟表,访问被删除的行.
2. old中的值全部都是只读的,不能更新.

### update触发器

1. 在update触发器代码中,可以引用一个名为old的虚拟表访问以前的值,引用一个名为new的虚拟表访问新更新的值.
2. 在before update触发器中,new中的值可能也被更新,允许更改将要用于update语句中的值.
3. old中的值全部是只读的,不能更新.

### 关于触发器的进一步介绍

1. 应该用触发器来保证数据的一致性(大小写,格式等),在触发器中执行这种类型的处理的优点是它总是进行这种处理,而且是透明地进行,与客户机应用无关.
2. 触发器的一种非常有意义的使用是创建审计跟踪,把更改记录到另一个表非常容易.

# 管理事务处理

## 事务处理

1. 事务指一组SQL语句.
2. 回退指撤销指定SQL语句的过程.
3. 提交指将未存储的SQL语句结果写入数据库表.
4. 保留点指事务处理中设置的临时占位符,你可以对它发布回退.

## 控制事务处理

```mysql
start transaction
```

### 使用rollback

rollback只能在一个事务处理内使用,在执行一条start transaction命令之后.事务处理用来管理insert,update和delete语句,你不能回退其他语句.

### 使用commit

### 使用保留点

为了支持回退部分事务处理,必须能在事务处理块中合适的位置放置占位符.

```mysql
savepoint s

rollback to s
```

保留点越多越好,你就越能按自己的意愿灵活地进行回退.

保留点在事务处理完成(执行一条rollback或commit)后自动释放,也可以用release savepoint明确地释放保留点.

### 更改默认的提交行为

默认的MySQL行为是自动提交所有更改,换句话说,任何时候你执行一条MySQL语句,该语句实际上都是针对表执行的,而且所做的更改立即生效,为指示MySQL不自动提交更改,需要使用以下语句.

```mysql
set autocommit=1
```

autocommit标识是针对每个连接而不是服务器的.

# 全球化和本地化

## 字符集和校对顺序

1. 字符集为字母和符号的集合.
2. 编码为某个字符集成员的内部表示.
3. 校对为规定字符如何比较的指令. 

使用字符集和校对的决定在服务器,数据库和表级进行.

## 使用字符集和校对顺序

```mysql
show character set

show collation
```

有的字符集具有不止一种校对,而且许多校对出现两次,一次区分大小写,由\_cs表示,一次不区分大小写,由\_ci表示.

```mysql
show variables like 'character%'

show variables like 'collation%
```

实际上,字符集很少是服务器范围,甚至数据库范围的设置,不同的表,甚至不同的列都可能需要不同的字符集,而且两者都可以在创建表时指定.

```mysql
create table t(
	i int character set latin1 
    	collate latin1_general_ci,
    n int
)default character set hebrew
collate hebrew_general_ci
```

校对在对用order by子句检索出来的数据排序时起重要的作用,如果需要用与创建表时不同的校对顺序排序特定的select语句.

```mysql
select * from t
order by i collate latin1_general_cs
```

collate还可以用与group by,having,聚集函数,别名等.

# 安全管理

## 管理用户

用户账号和信息存储在名为mysql的数据库中,一般不需要直接访问mysql数据库和表,但有时需要直接访问,需要直接访问它的时机之一是在需要获得所有用户账号列表时.

```mysql
use mysql
select user from user
```

### 创建用户账号

```mysql
create user u identified by 'p@$$w0rd'

rename user u to u1
```

### 删除用户账号

```msyql
drop user u
```

### 设置访问权限

```mysql
show grants for u@%
```

用户定义为user@host,MySQL的权限用用户名和主机名结合定义,如果不指定主机名,则使用默认的主机名%(授予用户访问权限而不管主机名).

usage表时没有权限.

```mysql
grant select on db.* to u

revoke select on db.* from u
```

grant和revoke可在几个层次上控制访问权限.

1. 整个服务器,使用grant all和revoke all.
2. 整个数据库,使用on db.*.
3. 特定的表,使用on db.t.
4. 特定的列.
5. 特定的存储过程.

使用grant和revoke时,用户账号必须存在,但对所涉及的对象没有这个要求,当某个数据库或表被删除时,相关的访问权限仍然存在.

```mysql
grant select,insert on db.* to u
```

### 更改口令

```mysql
set password for u=password('p@$$w0rd')

-- 当前登陆用户
set password=password('p@$$w0rd')
```

# 数据库维护

MySQL数据库是基于磁盘的文件,普通的备份系统和例程就能备份MySQL的而数据,但是,由于这些文件总是处于打开和使用状态,普通的文件副本备份不一定总是有效.

1. 使用命令行使用程序mysqldump转储所有数据库内容到某个外部文件,在进行常规备份前这个使用程序应该正常运行,以便能正确地备份转储文件.
2. 可用命令行使用程序mysqlhotcopy从一个数据库复制所有数据,并非所有数据库引擎都支持这个实用程序.
3. 可以使用MySQL的back table或select into outfile转储所有数据到某个外部文件,这两条语句都接受将要创建的系统文件名,数据可以用restore table来复原.

首先刷新未写数据,为了保证所有数据被写到磁盘(包括索引数据),可能需要在进行备份前使用flush tables语句.

## 进行数据库维护

1. analyze table用来检查表键是否正确.
2. check table用来针对许多问题对表进行检查,在MyISAM表上还对索引进行检查.
3. 如果MyISAM表访问产生不正确和不一致的结果,可能需要用repair table来修复相应的表,这条语句不应该经常使用,如果需要经常使用,可能会有更大的问题要解决.
4. 如果从一个表中删除大量数据,应该使用optimize table来收回所用的空间,从而优化表的性能.

## 诊断启动问题

通过在命令行上手动执行mysqld启动.

## 查看日志文件

1. 错误日志,它包含启动和关闭问题以及任意关键错误的细节,日志通常名为hostname.err,位于data目录中.
2. 查询日志,它记录所有MySQL活动,在诊断问题时非常有用,此日志文件可能会很快变得非常大,因此不应该长期使用它,此日志通常名为hostname.log.
3. 二进制日志,它记录更新过数据或者可能更新过数据的所有语句,此日志通常名为hostname-bin.
4. 缓慢查询日志,此日志记录执行缓慢的任何查询,此日志通常名为hostname-slow.log.

# 改善性能

1. 总是有不止一种方法编写同一条select语句,应该试验联结,并,子查询等,找出最佳的方法.
2. 使用explain语句让MySQL解释它讲如何执行一条select语句.
3. 一般来说,存储过程执行得比一条一条地执行其中得各条MySQL语句快.
4. 有的操作(包括insert)支持一个可选的delayed关键字,如果使用它,将把控制立即返回给调用程序,并且一旦有可能就实际执行该操作.
5. 在导入数据时,应该关闭自动提交,你可能还想删除索引(包括fulltext索引),然后在导入完成后再重建它们.
6. 如果一个简单的where子句返回结果所花的时间太长,则可以断定其中使用的列或几个列就是需要索引的对象.
7. 通过使用多条select语句和联结它们的union语句代替一系列复杂的or条件,可以看到极大的性能改进.
8. 索引改善数据检索的性能,但损害数据插入,删除和更新的性能.
9. like很慢,一般来说,最好是使用fulltex而不是like.

# MySQL数据类型

## 串数据类型

1. char.
2. varchar.
3. tinytext.
4. mediumtext.
5. text.
6. longtext.
7. enum.
8. set.

## 数值数据类型

1. float.
2. double.
3. tinyint.
4. smallint.
5. mediumint.
6. int.
7. bigint.
8. decimal.
9. boolean.
10. bit.
11. real.

## 日期和时间数据类型

1. date.
2. time.
3. year.
4. timestamp.
5. datetime.

## 二进制数据类型

1. tinyblob.
2. blob.
3. mediumblob.
4. longblob.