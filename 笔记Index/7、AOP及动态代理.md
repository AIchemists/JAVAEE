## AOP及动态代理
### 1	AOP是什么
AOP：全称是Aspect Oriented Programming，即面向切面编程。就是在多个纵向的流程中将相同的横切逻辑代码抽取出来，将横切逻辑代码与业务逻辑代码分离，解决代码重复和耦合的问题，提高程序的可重用性。  
简单的说它就是把我们程序重复的代码抽取出来，在需要执行的时候，使用动态代理的技术，在不修改源码的基础上，对我们的已有方法进行增强。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/3-1.png)    
**作用：** 在程序运行期间，不修改源码对已有方法进行增强。  
**优势：** 减少重复代码、提高开发效率、维护方便。  
AOP术语：  
 - Joinpoint(连接点): 被拦截到的方法。  
 - Pointcut(切入点): 我们对其进行增强的方法。  
 - Advice(通知/增强): 对切入点进行的增强操作。  
 
包括前置通知,后置通知,异常通知,最终通知,环绕通知  
 - Weaving(织入): 是指把增强应用到目标对象来创建新的代理对象的过程。  
 - Aspect(切面): 是切入点和通知的结合。  
 
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/3-2.png)   
### 2 AOP解决问题实例
```java
	@Override
	public void saveAccount(Account account) {
		try {
			TransactionManager.beginTransaction();
			accountDao.save(account);		// 唯一的一行业务代码
			TransactionManager.commit();
		} catch (Exception e) {
			TransactionManager.rollback();
			e.printStackTrace();
		}finally {
			TransactionManager.release();
		}
	}
	
	@Override
	public void updateAccount(Account account) {
		try {
			TransactionManager.beginTransaction();
			accountDao.update(account);		// 唯一的一行业务代码
			TransactionManager.commit();
		} catch (Exception e) {
			TransactionManager.rollback();
			e.printStackTrace();
		}finally {
			TransactionManager.release();
		}
	}
	
	@Override
	public void deleteAccount(Integer accountId) {
		try {
			TransactionManager.beginTransaction();
			accountDao.delete(accountId);	// 唯一的一行业务代码
			TransactionManager.commit();
		} catch (Exception e) {
			TransactionManager.rollback();
			e.printStackTrace();
		}finally {
			TransactionManager.release();
		}
	}
```
在上述代码中有两个问题:  
1、业务层方法变得臃肿了,里面充斥着很多重复代码。  
2、业务层方法和事务控制方法耦合了. 若提交、回滚、释放资源中任何一个方法名变更,都需要修改业务层的代码。  
可以用动态代理来解决这些问题。  

### 3 动态代理
动态代理：  
 - 特点：字节码随用随创建，随用随加载。它与静态代理的区别也在于此。因为静态代理是字节码一上来就 创建好，并完成加载。 装饰者模式就是静态代理的一种体现。  
 - 作用：在不修改源码的基础上对方法增强。  
 
常用的动态代理分为两种：  
1.	基于接口的动态代理,使用JDK 官方的 Proxy 类,要求被代理者至少实现一个接口  
2.	基于子类的动态代理,使用第三方的 CGLib库,要求被代理类不能是final类  
#### 3.1基于接口的动态代理
如何创建代理对象：**使用Proxy类中的newProxyInstance方法**  
（1）涉及的类：jdk官方提供的 Proxy；  
（2）要求：被代理类必须至少实现一个接口，如果没有则不能使用  
（3）newProxyInstance方法的参数：  
  - classLoader类加载器：它是用于加载代理对象字节码的，和被加载对象使用相同的类加载器  
  - Class[] 字节码数组   
  - InvocationHandler 用于提供增强的代码，通常情况下时匿名内部类，但不必须  
   
写法：  
```java
    接口名 新对象名 = (接口名)Proxy.newProxyInstance(
    被代理的对象.getClass().getClassLoader(),	// 被代理对象的类加载器,固定写法
    被代理的对象.getClass().getInterfaces(),	// 被代理对象实现的所有接口,固定写法
    new InvocationHandler() {	// 匿名内部类,通过拦截被代理对象的方法来增强被代理对象
        /* 被代理对象的任何方法执行时,都会被此方法拦截到
        	其参数如下:
                proxy: 代理对象的引用,不一定每次都用得到
                method: 被拦截到的方法对象
                args: 被拦截到的方法对象的参数
        	返回值:
        		被增强后的返回值
		*/
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            if("方法名".equals(method.getName())) {
            	// 增强方法的操作
                rtValue = method.invoke(被代理的对象, args);
                // 增强方法的操作
                return rtValue;
            }          
        }
    });
```
#### 3.2 基于子类的动态代理
**基于子类的动态代理：**  
如何创建代理对象：使用Enhancer类中的create方法  
(1)涉及的类：Enhancer；提供者：cglib库  
(2)要求：被代理类不能是最终类  
(3)create()方法的参数：  
  - class：字节码  
  - 类加载器：它是用于加载代理对象字节码的，和被加载对象使用相同的类加载器  
  - Callback： 用于提供增强的代码，通常情况下时匿名内部类，但不必须。我们一般写的是该接口的子 接口实现类  
  
写法：  
```java
public class Client {
    public static void main(String[] args) {
        final Producer producer = new Producer();

        Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor() {
            /**
             * 执行被代理对象的任何方法都会经过该方法
             * @param proxy
             * @param method
             * @param args
             * @param methodProxy
             * @return
             * @throws Throwable
             */
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                Object returnValue = null;

                Float money = (Float)args[0];
                if("saleProduct".equals(method.getName())) {
                    returnValue = method.invoke(producer, money*0.8f);
                }
                return returnValue;
            }
        });
```

### 4 小结
在这一部分中，首先学习了什么是AOP,讨论了当前代码会出现的各种问题，并给出了非SpringAOP的解决方法，用动态代理来解决。同时，动态代理分为基于接口的动态代理与基于子类的动态代理。  