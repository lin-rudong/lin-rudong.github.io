---
title: MyBatis面试
date: 2018-10-08 15:37:20
tags: 
categories: MyBatis
---
# MyBatis如何防止注入？
* #{}是预编译处理，${}是字符串替换。
* #{}可以防止SQL注入。

# 类的属性和表的字段不一样？
1. 定义字段的别名为属性名。
2. 映射属性名和字段名。
```
<resultMap id="" type="">
    <id property="" column=""/>
    <result property="" column=""/>
</resultMap>
```

# 获取自动生成的键值？
```
<insert id="" parameterType="">
    <selectKey keyProperty="id" order="AFTER" resultType="int">
        select last_insert_id()
    </selectKey>
    insert into...
</insert>
```

# 在mapper中传递多个参数？
1. 使用占位符
```
<select id="" resultMap="">
    select * from t where id=#{0} and name=#{1}
</select>
```
```
public T f(@param("id") Integer id,@param("name") String name);

<select id="f" resultType="T">
    select * from t where id=#{id} and name=#{name}
</select>
```
2. 使用Map集合参数
3. 使用Bean参数

# 动态标签？
* 在Xml映射文件内，已标签的形式编写动态SQL，完成逻辑判断和动态拼接SQL的功能。
* 9种标签：
    1. trim
    2. where
    3. set
    4. foreach
    5. if
    6. choose
    7. when
    8. otherwise
    9. bind
* 原理：使用OGNL（Object-Graph Navigation Languag）计算表达式，根据表达式的值动态拼接SQL。

# MyBatis不同的Xml映射文件中，id重复？
* 定位=namespace+id
* 如果有namespace可以重复，如果没有配置namespace会导致覆盖。

# MyBatis是半自动的ORM映射工具？
* Hibernate是全自动ORM映射工具，查询关联对象或关联集合对象时，可以根据对象关系模型直接获取。
* MyBatis则需要手动编写SQL。

# 一个Xml映射文件，需要一个Dao接口对应？Dao接口的方法能重载吗？
* Dao接口，即Mapper接口，接口的全限名就是namespace，接口的方法名就是id，接口的参数就是传递给SQL的参数。
* Mapper接口没有实现类，当调用接口方法时，会定位一个MappedStatement。
* Dao接口里的方法不能重载。
* Dao接口的工作原理：JDK动态代理，MyBatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的SQL。

# 接口绑定的实现方式？
1. 注解绑定
2. Xml绑定

# MyBatis是如何分页的？分页插件的原理是什么？
* MyBatis使用RowBounds对象进行分页，RowBounds是针对ResultSet执行的内存分页，而非物理分页。
* 可以用SQL实现物理分页，也可以使用分页插件完成物理分页。
* 分页插件的基本原理:插件的拦截方法内拦截待执行的SQL，然后重写SQL，添加物理分页语句和物理分页参数。

# MyBatis插件运行原理？
* MyBatis仅可以编写针对ParameterHandler、ResultSetHandler、StatementHandler和Executor这4种接口的插件，MyBatis使用JDK的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能。

# MyBatis延迟加载的实现原理？
* MyBatis仅支持association和collection关联集合对象的延迟加载。
* 原理：使用cglib创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，如果当拦截器方法发现字段是null值，就会单独发送事先保存好的查询，然后保存到对象的字段中。

# MyBatis的Executor执行器？
* SimpleExecutor、ReuseExecutor和BatchExecutor
1. SimpleExecutor：每次执行一次update或select，开启一个Statement对象，用完立刻关闭Statement对象。
2. ReuseExecutor：执行update或select，以SQL作为key查找Statement对象，存在直接使用，不存在则创建，不关闭Statement对象而是在Map<String,Statement>内重复使用。
3. BatchExecutor：执行update（JDBC批处理不支持select），将sql添加到批处理中，等待统一执行executeBatch()。