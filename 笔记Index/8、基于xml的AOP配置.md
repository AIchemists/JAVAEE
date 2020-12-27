## 基于xml的AOP配置
在这一部分中，将会学习使用Spring的AOP来完成代理的功能。  
### 1 必要准备
使用了与springIOC例子中相同的实体类，业务层和持久层代码。  
同时，导入AOP的jar包。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/3-3.png)   
### 2配置通知类
```java
public class Logger {
    public  void printLog(){
        System.out.println("Logger类中的pringLog方法开始记录日志了。。。");
    }
}
```
该通知类用于打印日志：计划让其在切入点方法执行之前执行（切入点方法就是业务层方法）。  
### 3 配置xml文件
1、首先，在bean.xml中引入约束并将通知类注入Spring容器中  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">
	
    <!--通知类-->
	<bean id="logger" class=" com.itheima.utils.Logger"></bean>
</beans>
```
2、使用<aop:config>标签声明AOP配置,所有关于AOP配置的代码都写在<aop:config>标签内  
```xml
<aop:config>
	<!-- AOP配置的代码都写在此处 -->
</aop:config>
```
3、使用<aop:aspect>标签配置切面,其属性如下	  
 - id: 指定切面的id  
 - ref: 引用通知类的id  
 
```xml
<aop:config>
	<aop:aspect id="logAdvice" ref="logger">
    	<!--配置通知的类型要写在此处-->
    </aop:aspect>
</aop:config>
```
4、使用<aop:pointcut>标签配置切入点表达式,指定对哪些方法进行增强,其属性如下  
 - id: 指定切入点表达式的id  
 - expression: 指定切入点表达式  
 ```xml
<aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))">
</aop:pointcut>
```
5、使用<aop:xxx>标签配置对应类型的通知方法  
其属性如下:  
 - method: 指定通知类中的增强方法名  
 - ponitcut-ref: 指定切入点的表达式的id  
 - poinitcut: 指定切入点表达式  
 
其中pointcut-ref和pointref属性只能有其中一个
具体的通知类型:  
 - <aop:before>: 配置前置通知,指定的增强方法在切入点方法之前执行  
 - <aop:after-returning>: 配置后置通知,指定的增强方法在切入点方法正常执行之后执行  
 - <aop:after-throwing>: 配置异常通知,指定的增强方法在切入点方法产生异常后执行  
 - <aop:after>: 配置最终通知,无论切入点方法执行时是否发生异常,指定的增强方法都会最后执行  
 - <aop:around>: 配置环绕通知,可以在代码中手动控制增强代码的执行时机  
 
代码:   
```xml
	<aop:config>
	    <aop:aspect id="logAdvice" ref="logger">
	        <!--指定切入点表达式-->
	        <aop:pointcut expression="execution(* com.itheima.service.impl.*.*(..))" id="pt1"></aop:pointcut>
	        <!--配置各种类型的通知-->
	        <aop:before method="printLogBefore" pointcut-ref="pt1"></aop:before>
	        <aop:after-returning method="printLogAfterReturning" pointcut-ref="pt1"></aop:after-returning>
	        <aop:after-throwing method="printLogAfterThrowing" pointcut-ref="pt1"></aop:after-throwing>
	        <aop:after method="printLogAfter" pointcut-ref="pt1"></aop:after>
	    </aop:aspect>
	</aop:config>
```
### 4 环绕通知的配置
环绕通知是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。  
Spring框架为我们提供了一个接口：`ProceedingJoinPoint`。该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用。该方法与自己写Handler类的方法类似，较为灵活。  
 - ProceedingJoinPoint对象的getArgs()方法返回被拦截的参数  
 - ProceedingJoinPoint对象的proceed()方法执行被拦截的方法  
 
代码：  
```java
// 环绕通知方法,返回Object类型
public Object printLogAround(ProceedingJoinPoint pjp) {
    Object rtValue = null;
    try {
        Object[] args = pjp.getArgs(); //获取被拦截方法参数       
        printLogBefore();			// 执行前置通知
    	rtValue = pjp.proceed(args);// 执行被拦截方法
        printLogAfterReturn();		// 执行后置通知
    }catch(Throwable e) {
        printLogAfterThrowing();	// 执行异常通知
    }finally {
        printLogAfter();			// 执行最终通知
    }
    return rtValue;
}
```
同时在xml中，使用<aop:around>:配置环绕通知,可以在代码中手动控制增强代码的执行时机。  
```xml
<aop:aspect id="logAdvice" ref="logger">
    <aop:around method="aroundPrintLog" pointcut-ref="pt1"></aop:around>
</aop:aspect>
```
**注意：环绕通知一般单独使用**  

### 5切入点表达式
execution:匹配方法的执行(常用)   
```java
execution(表达式) 
```
表达式语法：execution([修饰符] 返回值类型 包名.类名.方法名(参数))   
**写法说明**   
全匹配方式：   
```java
public void com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account) 
```
访问修饰符可以省略  
``` java 
void com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account)
```
返回值可以使用*号，表示任意返回值  
```java 
* com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account) 
```
包名可以使用*号，表示任意包，但是有几级包，需要写几个  
```java
* * *.*.*.*.AccountServiceImpl.saveAccount(com.itheima.domain.Account) 
```
使用..来表示当前包，及其子包  
```java 
* com..AccountServiceImpl.saveAccount(com.itheima.domain.Account) 
```
类名可以使用*号，表示任意类   
```java 
* com..*.saveAccount(com.itheima.domain.Account) 
```
方法名可以使用*号，表示任意方法  
```java 
* com..*.*( com.itheima.domain.Account) 
```
参数列表可以使用*，表示参数可以是任意数据类型，但是必须有参数   
```java 
* com..*.*(*) 
```
参数列表可以使用..表示有无参数均可，有参数可以是任意类型   
```java
* com..*.*(..) 
```
全通配方式：  
```java
* *..*.*(..) 
```
**注意： 通常情况下，我们都是对业务层的方法进行增强，所以切入点表达式都是切到业务层实现类。**  
``` java
execution(* com.itheima.service.impl.*.*(..))
```
