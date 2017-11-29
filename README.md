# spring-aop
Spring aop 简单示例
第一种配置方式

　　基于xml方式配置

　　首先将service，dao注册到spring容器

　　

　　配置一下扫描包还是很方便的

　　接下来看下service

　　

复制代码
 1 package com.yangxin.core.service.impl;
 2 
 3 import org.springframework.beans.factory.annotation.Autowired;
 4 import org.springframework.stereotype.Service;
 5 
 6 import com.yangxin.core.dao.UserDao;
 7 import com.yangxin.core.pojo.User;
 8 import com.yangxin.core.service.UserService;
 9 
10 @Service
11 public class UserServiceImpl implements UserService {
12 
13     @Autowired
14     private UserDao userDao;
15     
16     @Override
17     public void addUser(User user) {
18         userDao.insertUser(user);
19         System.out.println("添加成功");
20     }
21 
22     @Override
23     public void deleteUser(String name) {
24         userDao.deteleUser(name);
25         System.out.println("删除成功");
26     }
27 
28 }
复制代码
要做的事情很简单，插入一条数据，删除一条数据

 

接下来看下切面代码

复制代码
 1 package com.yangxin.core.transaction;
 2 
 3 import org.aspectj.lang.ProceedingJoinPoint;
 4 
 5 import com.yangxin.core.pojo.User;
 6 
 7 public class TransactionDemo {
 8     
 9     //前置通知
10     public void startTransaction(){
11         System.out.println("begin transaction ");
12     }
13     
14     //后置通知
15     public void commitTransaction(){
16         System.out.println("commit transaction ");
17     }
18     
19     //环绕通知
20     public void around(ProceedingJoinPoint joinPoint) throws Throwable{
21         System.out.println("begin transaction");
22         
23         joinPoint.proceed();
24         
25         System.out.println("commit transaction");
26     }
27     
28 }
复制代码
然后看下这个切面在applicationContext.xml中是如何配置的



　

复制代码
 1 <aop:config>
 2         <aop:pointcut expression="execution(* com.yangxin.core.service.*.*.*(..))" id="p1" />
 3 
 4         <aop:aspect ref = "transactionDemo">
 5         
 6         <aop:before method="startTransaction" pointcut-ref="p1" />
 7         
 8         <aop:after-returning method="commitTransaction" pointcut-ref="p1"/>
 9         
10         </aop:aspect>
11     </aop:config>
复制代码
这里没有演示环绕通知

好了，运行测试代码

测试代码如下

复制代码
 1 　　@Test
 2     public void test1(){
 3         ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/applicationContext.xml");
 4         
 5         UserService userService = applicationContext.getBean(UserService.class);
 6         
 7         User user = new User();
 8         
 9         user.setAge(19);
10         user.setName("yangxin");
11                 
12         userService.addUser(user);
13         userService.deteleUser("yangxin");
1415         
16     }
复制代码
控制台输出如下

　　begin transaction

　　添加成功

　　commit transaction

　　begin transaction

　　删除成功

　　commit transaction

现在来测试一下环绕通知

修改一下applicationContext.xml中的配置切面那一部分

修改后的代码

复制代码
1 <aop:config>
2         <aop:pointcut expression="execution(* com.yangxin.core.service.*.*.*(..))" id="p1" />
3 
4         <aop:aspect ref = "transactionDemo">
5         
6         <aop:around method="around" pointcut-ref="p1"/>
7         
8         </aop:aspect>
9     </aop:config>
复制代码
运行测试代码

输出如下

begin transaction
添加成功
commit transaction
begin transaction
删除成功
commit transaction

 

好了，现在贴下如何用注解的方法

贴下基于注解的切面的代码

复制代码
 1 package com.yangxin.core.transaction;
 2 
 3 import org.aspectj.lang.ProceedingJoinPoint;
 4 import org.aspectj.lang.annotation.AfterReturning;
 5 import org.aspectj.lang.annotation.Around;
 6 import org.aspectj.lang.annotation.Aspect;
 7 import org.aspectj.lang.annotation.Before;
 8 import org.aspectj.lang.annotation.Pointcut;
 9 
10 @Aspect
11 public class TransactionDemo2 {
12     
13     @Pointcut(value="execution(* com.yangxin.core.service.*.*.*(..))")
14     public void point(){
15         
16     }
17     
18     @Before(value="point()")
19     public void before(){
20         System.out.println("transaction begin");
21     }
22     
23     @AfterReturning(value = "point()")
24     public void after(){
25         System.out.println("transaction commit");
26     }
27     
28     @Around("point()")
29     public void around(ProceedingJoinPoint joinPoint) throws Throwable{
30         System.out.println("transaction begin");
31         joinPoint.proceed();
32         System.out.println("transaction commit");
33         
34     }
35 }
复制代码
在applicationContext.xml中配置

1 <bean id = "transactionDemo2" class = "com.yangxin.core.transaction.TransactionDemo2" />
1 <aop:aspectj-autoproxy />
测试步骤和以上一致，这里就不贴了

 

完毕

 

记一下使用javaConfig配置方式的一些坑

 

以上 切面 不管是基于注解的还是基于xml配置的   这里把切面加载到容器中都是用xml配置bean的方式

 

如果用javaConfig方式的话

 

需要在配置类上加上 @EnableAspectJAutoProxy 注解

 

然后如果使用扫包的方式配置切面bean的话

切面上除了要加上@Aspect注解标识为这是一个切面bean之外

还需要在上面加上@component这个注解

很急很关键

排错的时候只关注切点表达式的问题了

当时一直在找切点表达式的问题

其实这个表达式写的是没有问题的

是配置切面bean的时候出的问题

深坑
