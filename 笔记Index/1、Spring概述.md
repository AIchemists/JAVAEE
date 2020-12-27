## Spring概述
### 1 Spring是什么
Spring 框架是一个分层架构，由 7 个定义良好的模块组成。Spring 模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式。Spring以IoC（Inverse Of Control：反转控制）和AOP（Aspect Oriented Programming：面向切面编程）为内核。  

### 2 程序的耦合与解耦
#### 2.1程序的耦合
耦合指的就是就是对象之间的依赖性。对象之间的耦合越高，维护成本越高。因此对象的设计应使类和构件之间的耦合最小。软件设计中通常用耦合度和内聚度作为衡量模块独立程度的标准。划分模块的一个准则就是高内聚低耦合。内聚是从功能角度来度量模块内的联系，一个好的内聚模块应当恰好做一件事。它描述的是模块内的功能联系。  
解耦：降低程序间的依赖关系。  
在实际开发中应该做到：编译期不依赖，运行时才依赖。  
**耦合实例1：**  
JDBC操作中注册驱动时,使用DriverManager的register方法的流程如下：  
```java
//1.注册驱动 
//DriverManager.registerDriver(new com.mysql.jdbc.Driver());
//2.获取连接 
//3.获取预处理 sql 语句对象 
//4.获取结果集 
//5.遍历结果集
```
如果用这样的方式，DriverManager类依赖了数据库的具体驱动类（MySQL），如果这时候更换了数据库类型，需要修改源码来重新数据库驱动。这显然是高耦合的。  
因此实际上使用jdbc 时，是通过反射来注册驱动的，代码如下：  
```java
Class.forName("com.mysql.jdbc.Driver");
```
此时的好处是，我们的类中不再依赖具体的驱动类，此时就算删除 mysql 的驱动 jar 包，依然可以编译。即使驱动类不存在,在编译时也不会报错,解决了编译器依赖。实际上，开发中此类名会从properties文件中读取。  
**耦合实例2：**  
在Web项目中,UI层,Service层,Dao层之间有着前后调用的关系。  
```java
public class AccountServiceImpl implements IAccountService 
{ 
private IAccountDao accountDao = new AccountDaoImpl(); 
}
```
业务层依赖持久层的接口和实现类,若编译时不存在没有持久层实现类,则编译将不能通过,这构成了编译期依赖。  

#### 2.2程序的解耦
解耦的思路: 使用反射来创建对象,而避免使用new关键字,并通过读取配置文件来获取要创建的对象全限定类名。  
在实际开发中可以把三层的对象的全类名都使用配置文件保存起来,当启动服务器应用加载的时候,创建这些对象的实例并保存在容器中. 在获取对象时,不使用new的方式,而是直接从容器中获取,这就是工厂设计模式。这个读取配置文件，创建和获取三层对象的类就是工厂。  
工厂负责给我们从容器中获取指定对象的类。这时候我们获取对象的方式发生了改变。  
之前我们在获取对象时，都是采用 new 的方式。是主动的。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/1-1.png)    
现在我们获取对象时，同时跟工厂（容器）获取，有工厂为我们查找或者创建对象，是被动的。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/1-2.png)   
这种被动接收的方式（由主动变被动）获取对象的思想就是控制反转IOC，它是spring框架的核心之一。  

### 3 Spring的IOC
IoC对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系,而不是由程序直接控制。  
**控制 ：**指的是对象创建（实例化、管理）的权力  
**反转 ：**控制权交给外部环境（Spring 框架、IoC 容器）  
A对象要使用B对象，就主动创建一个B对象，那么A就对B产生了依赖，也就是A和B之间存在一种耦合关系，而在spring中，创建对象B的工作是由Spring来做的，Spring创建好B对象，然后存储到一个容器里面，当A对象需要使用B对象时，Spring就从存放对象的那个容器里面取出A要使用的那个B对象，然后交给A对象使用，至于Spring是如何创建那个对象，以及什么时候创建好对象的，A对象不需要关心这些细节问题
所以控制反转IoC(Inversion of Control)是说创建对象的控制权进行转移，传统创建对象的主动权是由自己把控的，而现在这种权力转移到第三方，即spring，spring控制所有对象，程序只需要被动的接受就行了。  