## Spring IOC的xml配置实现
### 1 准备工作
创建maven项目,配置其pom.xml如下:  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.maoritian</groupId>
    <artifactId>learnspring</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
    	<!-- 引入-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```
### 2三层接口类和实现类
**业务层：**
```java
public interface IAccountService
{ 
void saveAccount(); 
}
public class AccountServiceImpl implements IAccountService 
{ 
private IAccountDao accountDao = new AccountDaoImpl();//此处的依赖关系有待解决 @Override 
public void saveAccount() 
{ 
accountDao.saveAccount(); 
} 
}
```
可以看到，此处的业务层类与持久层类IAccountDao存在耦合关系
**持久层:**
```java
public interface IAccountDao 
{
void saveAccount();
}
public class AccountDaoImpl implements IAccountDao 
{ 
@Override 
public void saveAccount() 
{ 
System.out.println("保存了账户");
}
}
```
可以看到，此时需要我们解决业务层与逻辑层类的耦合关系，添加xml配置文件来进行解耦。  
### 3 xml配置文件
首先，创建bean.xml文件：  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/1-3.png)     
给xml文件导入约束：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
   ```
把对象的创建交给spring来管理：  
```xml
     <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>
     <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl"></bean>
 </beans>
```
在这里先后定义了accountService和accountDao两个类的容器，其中  
- bean 标签：用于配置让 spring 创建对象，并且存入 ioc 容器之中   
- id 属性：对象的唯一标识。   
- class 属性：指定要创建对象的全限定类名  
这样一来，就完成了Spring IOC的xml文件的配置。  
整个项目结构如下：  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/1-4.png)   
### 4通过核心容器创建对象
在代码中创建核心容器，通过核心容器的getBean()方法获取具体对象  
```java
        // 获取核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        // 根据id获取bean对象
        IAccountService as = (IAccountService) ac.getBean("accountService");
        IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);
```
在这里，我们的核心容器实际上相当于一个map。  
常用的容器有3中：  
`ClassPathXmlApplicationContext`,`FileSystemXmlApplicationContext`,`AnnotationConfigApplicationContext`.  
1、`ClassPathXmlApplicationContext`: 它是从类的根路径下加载配置文件（推荐）。  
2、`FileSystemXmlApplicationContext`: 它是从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。  
3、`AnnotationConfigApplicationContext`: 读取注解创建容器，当我们使用注解配置容器对象时，需要使用此类来创建 spring 容器。它用来读取注解。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/1-5.png)   
BeanFactory和ApplicationContext 的区别：  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/1-6.png)   
`ApplicationContext`：它在创建核心容器时，创建对象采取的策略是采用立即加载的方式，也就是说，只要一读取完配置文件就马上创建配置文件中配置的对象。  
- 单例对象适用  
- 开发中常采用此接口  
`BeanFactory`:它在构建核心容器时，创建对象的策略是采用延迟加载的方式，什么时候获取id对象了，什么时候就创建对象。  
- 多例对象适用  
### 5 小结
在这一部分中，学习了用xml实现springIOC的方法与步骤，从创建bean.xml、导入约束，定义类的容器以及利用容器创建类、容器的关系等等，主要学习的是xml的基本使用方法。  