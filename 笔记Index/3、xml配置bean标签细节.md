## xml配置bean标签细节内容
### 1 bean 标签
**作用：** 用于配置对象让 spring 来创建的。   
默认情况下它调用的是类中的无参构造函数。如果没有无参构造函数则不能创建成功。  
**属性：**   
+ id：给对象在容器中提供一个唯一标识。用于获取对象。  
+ class：指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数。  
+ scope：指定对象的作用范围。  
  * singleton :默认值，单例的. 
  * prototype :多例的. 
  * request :WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到 request 域中. 
  * session :WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到 session 域中. 
  * global session :WEB 项目中,应用在 Portlet 环境.如果没有 Portlet 环境那么 globalSession 相当于 session. 
+ init-method：指定类中的初始化方法名称。
+ destroy-method：指定类中销毁方法名称。  
### 2 bean的作用范围和生命周期
1、单例对象: scope="singleton"  
**作用范围:** 每个应用只有一个该对象的实例,它的作用范围就是整个应用  
**生命周期: **单例对象的创建与销毁 和 容器的创建与销毁时机一致  
**对象出生:** 当应用加载,创建容器时,对象就被创建  
**对象活着: **只要容器存在,对象一直活着  
**对象死亡:** 当应用卸载,销毁容器时,对象就被销毁  
2、	多例对象: scope="prototype"  
**作用范围:** 每次访问对象时,都会重新创建对象实例  
**生命周期:** 多例对象的创建与销毁时机不受容器控制  
**对象出生:** 当使用对象时,创建新的对象实例  
**对象活着: **只要对象在使用中,就一直活着  
**对象死亡: **当对象长时间不用时,被 java 的垃圾回收器回收了  
### 3 实例化 Bean 的三种方式
1.使用默认无参构造函数创建对象: 默认情况下会根据默认无参构造函数来创建类对象,若Bean类中没有默认无参构造函数,将会创建失败.  
```xml
<bean id="accountService" 
	class="cn.itheima.service.impl.AccountServiceImpl"></bean>
```

2、使用静态工厂的方法创建对象:  
创建静态工厂如下:  
```java
// 静态工厂,其静态方法用于创建对象
public class StaticFactory {
	public static IAccountService createAccountService(){
		return new AccountServiceImpl();
	}
}
```
使用StaticFactory类中的静态方法createAccountService创建对象,涉及到<bean>标签的属性:  
1、id属性: 指定对象在容器中的标识,用于从容器中获取对象  
2、class属性: 指定静态工厂的全类名  
3、factory-method属性: 指定生产对象的静态方法  
```xml
<bean id="accountService"
	class="cn.maoritian.factory.StaticFactory"
	factory-method="createAccountService"></bean>
```
实际上，类的构造函数也是静态方法,因此默认无参构造函数也可以看作一种静态工厂方法  
3、使用实例工厂的方法创建对象  
创建实例工厂如下:  
```java
public class InstanceFactory {
	public IAccountService createAccountService(){
		return new AccountServiceImpl();
	}
}
```
先创建实例工厂对象instanceFactory,通过调用其createAccountService()方法创建对象,涉及到<bean>标签的属性:  
1、	factory-bean属性: 指定实例工厂的id  
2、factory-method属性: 指定实例工厂中生产对象的方法  
```xml
<bean id="instancFactory" class="cn.itheima.factory.InstanceFactory"></bean>
<bean id="accountService"
	factory-bean="instancFactory"
	factory-method="createAccountService"></bean>
```
### 4依赖注入
在3.3中学习了spring的实例创建方法，此外，Spring还通过依赖注入来创建存在耦合的类和属性。  
#### 4.1 Spring中的依赖注入
依赖注入(Dependency Injection)是spring框架核心ioc的具体实现.  
通过控制反转,我们把创建对象托管给了spring,但是代码中不可能消除所有依赖,例如:业务层仍然会调用持久层的方法,因此业务层类中应包含持久化层的实现类对象。  
我们等待框架通过配置的方式将持久层对象传入业务层,而不是直接在代码中new某个具体的持久化层实现类,这种方式称为依赖注入。  
#### 4.2使用构造函数注入
通过类默认的构造函数来给创建类的字段赋值,相当于调用类的构造方法。  
涉及的标签: `<constructor-arg>`用来定义构造函数的参数,其属性可大致分为两类:  
1.	寻找要赋值给的字段  
 -	index: 指定参数在构造函数参数列表的索引位置  
 -	type: 指定参数在构造函数中的数据类型  
 -	name: 指定参数在构造函数中的变量名,最常用的属性  
2.	指定赋给字段的值  
 -	value: 给基本数据类型和String类型赋值  
 -	ref: 给其它Bean类型的字段赋值,ref属性的值应为配置文件中配置的Bean的id  
实体类代码:  
  ```java
public AccountServiceImpl(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
  ```
xml配置:   
```xml
<!-- 使用Date类的无参构造函数创建Date对象 -->
<bean id="now" class="java.util.Date" scope="prototype"></bean>

<bean id="accountService" class=" com.itheima.service.impl.AccountServiceImpl">
	<constructor-arg name="name" value="myname"></constructor-arg>
	<constructor-arg name="age" value="18"></constructor-arg>
	<!-- birthday字段为已经注册的bean对象,其id为now -->
	<constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
```

#### 4.3 set 方法注入
在类中提供需要注入成员属性的set方法,创建对象只调用要赋值属性的set方法.  
涉及的标签: `<property>`,用来定义要调用set方法的成员. 其主要属性可大致分为两类:  
1、指定要调用set方法赋值的成员字段  
 - name：要调用set方法赋值的成员字段  
 
2、指定赋给字段的值  
 - value: 给基本数据类型和String类型赋值  
 - ref: 给其它Bean类型的字段赋值,ref属性的值应为配置文件中配置的Bean的id  
实体类代码：  
```java
public void setName(String name) {
		this.name = name;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}
```
xml配置：
```xml
<!-- 使用Date类的无参构造函数创建Date对象 -->
<bean id="now" class="java.util.Date" scope="prototype"></bean>

<bean id="accountService" class=" com.itheima.service.impl.AccountServiceImpl ">
	<property name="name" value="myname"></property>
	<property name="age" value="21"></property>
	<!-- birthday字段为已经注册的bean对象,其id为now -->
	<property name="birthday" ref="now"></property>
</bean>
```
#### 4.4注入集合字段
集合字段及其对应的标签按照集合的结构分为两类: 相同结构的集合标签之间可以互相替换，本次学习了数组，List,Set,Map,Properties。  
**只有键的结构:**   
 - 数组字段: <array>标签表示集合,<value>标签表示集合内的成员.  
 - List字段: <list>标签表示集合,<value>标签表示集合内的成员.  
 - Set字段: <set>标签表示集合,<value>标签表示集合内的成员.  
其中<array>,<list>,<set>标签之间可以互相替换使用.  
  
**键值对的结构:**  
 - Map字段: <map>标签表示集合,<entry>标签表示集合内的键值对,其key属性表示键,value属性表示值.  
 - Properties字段: <props>标签表示集合,<prop>标签表示键值对,其key属性表示键,标签内的内容表示值.  
**其中<map>,<props>标签之间,<entry>,<prop>标签之间可以互相替换使用.**  

实体类的字段：  
```java
private String[] myArray;
	private List<String> myList;
	private Set<String> mySet;
	private Map<String,String> myMap;
	private Properties myProps;
  ```
xml配置：  
```xml
<bean id="accountService" class=" com.itheima.service.impl.AccountServiceImpl ">
<!--给数组注入数据--> 
	<property name="myStrs">
		<array>
			<value>value1</value>
			<value>value2</value>
			<value>value3</value>
		</array>
	</property>
<!--注入 list 集合数据-->
	<property name="myList">
		<list>
			<value>value1</value>
			<value>value2</value>
			<value>value3</value>
		</list>
	</property>
<!--注入 set 集合数据 --->
	<property name="mySet">
		<set>
			<value>value1</value>
			<value>value2</value>
			<value>value3</value>
		</set>
	</property>
<!--注入 Map 数据-->
	<property name="myMap">
		<map>
			<entry key="key1" value="value1"></entry>
			<entry key="key2">
				<value>value2</value>
			</entry>
			
		</map>
	</property>
<!--注入 properties 数据-->
	<property name="myProps">
		<props>
			<prop key="key1">value1</prop>
			<prop key="key2">value2</prop>
		</props>
	</property>
</bean>
```
**注意：在注入集合数据时，只要结构相同，标签可以互换。**  
其中，List 结构的:array,list,set   
Map 结构的:map,entry,props,prop  

### 5 小结
在这一部分中，学习了使用bean标签具体实例化一个类的各种细节。首先，学习了bean标签内部的一些基本标签，声明该实例的注入属性、作用范围和生命周期等等。接着学习了实例化一个bean时的三种方法，以及实体类中所需要的函数：构造函数等等。最后，通过依赖注入我们可以注入实体类中的字段/属性的三种方法，同时实体类中也需要像一个的构造函数、set方法。

