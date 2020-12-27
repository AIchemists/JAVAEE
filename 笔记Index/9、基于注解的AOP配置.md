## 基于注解的 AOP 配置
### 1 基本准备
基于注解的AOP仍然是使用IOC的实体类与注解，不同的是需要在配置文件中先导入context的名称空间。同时，在xml中指定创建容器要扫描的包。  
```xml
< context:component-scan base-package="com.itheima"></context:component-scan>
```
### 2 通过注解进行通知配置
#### 2.1用于声明切面的注解
```java
@Aspect: 声明当前类为通知类,该类定义了一个切面.相当于xml配置中的<aop:aspect>标签
@Component("logger")
@Aspect
public class Logger {
    // ...
}
```
#### 2.2用于声明通知的注解
`@Before`: 声明该方法为前置通知.相当于xml配置中的<aop:before>标签  
`@AfterReturning`: 声明该方法为后置通知.相当于xml配置中的<aop:after-returning>标签  
`@AfterThrowing`: 声明该方法为异常通知.相当于xml配置中的<aop:after-throwing>标签  
`@After`: 声明该方法为最终通知.相当于xml配置中的<aop:after>标签  
 
属性:  
 - value: 用于指定切入点表达式或切入点表达式的引用  
```java
	@Component("logger")
	@Aspect	//表示当前类是一个通知类
	public class Logger {
	    // 配置前置通知
		@Before("execution(* cn. itheima.service.impl.*.*(..))")
	    public void printLogBefore(){
	        System.out.println("前置通知Logger类中的printLogBefore方法开始记录日志了。。。");
	    }
	    // 配置后置通知
	    @AfterReturning("execution(* cn.maoritian.service.impl.*.*(..))")
	    public void printLogAfterReturning(){
	        System.out.println("后置通知Logger类中的printLogAfterReturning方法开始记录日志了。。。");
	    }  
	    // 配置异常通知
		@AfterThrowing("execution(* cn. itheima.service.impl.*.*(..))")
	    public void printLogAfterThrowing(){
	        System.out.println("异常通知Logger类中的printLogAfterThrowing方法开始记录日志了。。。");
	    }
	    // 配置最终通知
	    @After("execution(* cn. itheima.service.impl.*.*(..))")
	    public void printLogAfter(){
	        System.out.println("最终通知Logger类中的printLogAfter方法开始记录日志了。。。");
	    }
	}
```
#### 2.3用于指定切入点表达式的注解
`@Pointcut`: 指定切入点表达式,其属性如下:  
 - value: 指定表达式的内容  
 
@Pointcut注解没有id属性,通过调用被注解的方法获取切入点表达式。  
```java
// 配置切入点表达式
    @Pointcut("execution(* cn.itheima.service.impl.*.*(..))")
    private void pt1(){} 
    
    // 通过调用被注解的方法获取切入点表达式
	@Before("pt1()")
    public void printLogBefore(){
        System.out.println("前置通知Logger类中的printLogBefore方法开始记录日志了。。。");
    }
```
#### 2.5 环绕通知的注解
`@Around`: 声明该方法为环绕通知,相当于xml配置中的<aop:around>标签。  
代码:  
```java
// 配置环绕通知
    @Around("execution(* cn. itheima.service.impl.*.*(..))")
    public Object aroundPringLog(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try{
            Object[] args = pjp.getArgs();	
			printLogBefore();				// 执行前置通知
            rtValue = pjp.proceed(args);	// 执行切入点方法
			printLogAfterReturning();		// 执行后置通知
            return rtValue;
        }catch (Throwable t){
            printLogAfterThrowing();		// 执行异常通知
            throw new RuntimeException(t);
        }finally {
            printLogAfter();				// 执行最终通知
        }
    }
```
### 3 纯注解配置AOP
在Spring配置类前添加`@EnableAspectJAutoProxy`注解,可以使用纯注解方式配置AOP。  
```java
@Configuration
@ComponentScan(basePackages="cn.itheima")
@EnableAspectJAutoProxy			// 允许AOP
public class SpringConfiguration {
    // 具体配置
    //...
}
```
**注意：在使用注解配置AOP时,会出现一个bug。四个通知的调用顺序依次是:前置通知,最终通知,后置通知。这会导致一些资源在执行最终通知时提前被释放掉了,而执行后置通知时就会出错。** 