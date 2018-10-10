---
title: Spring实战
categories: Spring
abbrlink: 36040
date: 2018-10-08 16:11:31
tags:
---
# Spring实战

## ApplicationContext：基于BeanFactory

## bean的生命周期
1. 实例化
2. 注入和Aware接口
3. BeanPostProcessor接口
4. InitializingBean接口或init-method方法
5. BeanPostProcessor接口
6. 准备就绪和一直驻留ApplicationContext
7. DisposableBean接口或destroy-method方法

## 装配Bean
* 隐式组件扫描和自动装配
    * @Configuration
    * @ComponentScan @ComponentScan() @ComponentScan(basePackages=) @ComponentScan(basePackages={}) @ComponentScan(basePackageClasses={}) [类坐在的包]
    * @Component @Component()
    * @Autowired @Autowired(required=true) [构造器 Setter方法 字段（安全管理必须允许通过反射访问私有字段）]
* 显式Java
    * @Configuration
    * @Bean @Bean(name=)
* 显式XML

### JavaConfig引用其他配置
* @Import(Class) @Import({})
* @ImportResource("xml")

### XML引用其他配置

## 高级装配
* 运行时值注入
    * 占位符 "${}"
        * @PropertySource()
        * @Value(占位符) 必须配置PropertySourcesPlaceholderConfigurer
    * SpEL Spring表达式语言 "#{}" #{3.14} #{'hello'} #{true}
        1. 使用bean的ID id.field id.method() ?.[可null]
        2. 调用方法和访问字段 T()[类]
        3. 算术^、关系、逻辑and or not |、条件？：[可判断null true没有表示不变]
        4. 正则表达式匹配 matches
        5. 集合操作[] .?[]过滤 .^[] .$[] .![]投影

## junit测试
* @RunWith(SpringJUnit4ClassRunner.class) 自动创建应用上下文
* ContextConfiguration(classes=) 



## AOP 切面=切点+增强
* EnableAspectJAutoProxy
* @Aspect
* @Pointcut


### 切点 选择方法(连接点)
* execution(* pkg.class.method(..)) && within(pkg.*) && !bean('id')
* execution(* pkg.class.method(int)) && args(增强参数)
* 指示器
    1. args()
    2. @args()
    3. execution()
    4. this()
    5. target()
    6. @target()
    7. within()
    8. @within()
    9. @annotation


#### 增强
* @Before()
* @After()
* @AfterReturning()
* @AfterThrowing()
* @Around()
```
@Around()
method(ProceedingJoinPoint jp){
    ...
    jp.proceed();
    ...
}
```

