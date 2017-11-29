# spring-aop
Spring aop 简单示例
第一种配置方式

　　基于xml方式配置

　　首先将service，dao注册到spring容器,配置一下扫描包

　　接下来看下service

package com.yangxin.core.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.yangxin.core.dao.UserDao;
import com.yangxin.core.pojo.User;
import com.yangxin.core.service.UserService;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;
    
    @Override
    public void addUser(User user) {
        userDao.insertUser(user);
        System.out.println("添加成功");
    }

    @Override
    public void deleteUser(String name) {
        userDao.deteleUser(name);
        System.out.println("删除成功");
    }

}

接下来看下切面代码

package com.yangxin.core.transaction;

import org.aspectj.lang.ProceedingJoinPoint;

import com.yangxin.core.pojo.User;

public class TransactionDemo {
    
    //前置通知
    public void startTransaction(){
        System.out.println("begin transaction ");
    }
    
    //后置通知
    public void commitTransaction(){
        System.out.println("commit transaction ");
    }
    
    //环绕通知
    public void around(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("begin transaction");
        
        joinPoint.proceed();
        
        System.out.println("commit transaction");
    }
    
}

然后看下这个切面在applicationContext.xml中是如何配置的

<aop:config>
        <aop:pointcut expression="execution(* com.yangxin.core.service.*.*.*(..))" id="p1" />

        <aop:aspect ref = "transactionDemo">
        
        <aop:before method="startTransaction" pointcut-ref="p1" />
        
        <aop:after-returning method="commitTransaction" pointcut-ref="p1"/>
        
        </aop:aspect>
    </aop:config>
    

这里没有演示环绕通知

好了，运行测试代码

测试代码如下

@Test
    public void test1(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/applicationContext.xml");
        
        UserService userService = applicationContext.getBean(UserService.class);
        
        User user = new User();
        
        user.setAge(19);
        user.setName("yangxin");
                
        userService.addUser(user);
        userService.deteleUser("yangxin");
        
    }
    
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

<aop:config>
        <aop:pointcut expression="execution(* com.yangxin.core.service.*.*.*(..))" id="p1" />

        <aop:aspect ref = "transactionDemo">
        
        <aop:around method="around" pointcut-ref="p1"/>
        
        </aop:aspect>
    </aop:config>
    
    
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

package com.yangxin.core.transaction;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class TransactionDemo2 {
    
    @Pointcut(value="execution(* com.yangxin.core.service.*.*.*(..))")
    public void point(){
        
    }
    
    @Before(value="point()")
    public void before(){
        System.out.println("transaction begin");
    }
    
    @AfterReturning(value = "point()")
    public void after(){
        System.out.println("transaction commit");
    }
    
    @Around("point()")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("transaction begin");
        joinPoint.proceed();
        System.out.println("transaction commit");
        
    }
}

在applicationContext.xml中配置

<bean id = "transactionDemo2" class = "com.yangxin.core.transaction.TransactionDemo2" />
<aop:aspectj-autoproxy />



