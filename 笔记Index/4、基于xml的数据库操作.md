## 基于XML配置的IOC
在这两个部分的内容中，使用DBAssit作为持久层解决方案，使用c3p0数据源。  
### 1 必要的准备
首先，由于使用了dbAssit等等工具，需要导入额外的jar包。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/2-1.png)    
接着，需要为数据库操作的持久层、逻辑层以及实体类编写一些必要的类，如  
`Account`类作为实体类是被操作的对象；  
`IAccountDao`是持久层接口，声明CRUD操作；  
`AccountDaoImpl`类是持久层实际操作的类，实现CRUD方法，在这个类中有一个dbAssit类作为属性，来进行数据库连接。  
`AccountServiceImpl`类来封装持久层接口，进行数据库操作。  
	大致项目结构如下。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/2-2.png)    
使用ioc完成CRUD操作所需要的几个类如上所述，接下来就是xml的配置部分。  

### 2 xml配置
#### 2.1数据源的配置
Java需要配置数据源来进行数据库的连接和访问，因此首先需要在xml中配置一个数据源类。具体代码如下所示。  
```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <!--连接数据库的必备信息-->
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/eesy"></property>
    <property name="user" value="root"></property>
    <property name="password" value="1234"></property>
</bean>
```
数据源中包含了数据库连接的进本信息，如数据库用户名密码、连接池管理等等内容。  

#### 2.2dbAssit的配置
dbAssit是导入的jar包中的类，数据源是一个属性。dbAssit应该是用来负责数据库的连接与操作的（其实也可以用JDBCTemplate来代替），具体代码如下。  
```xml
<bean id="dbAssit" class="comitheima.dao.dbassit.DBAssit">
    <!--注入数据源-->
    <property name="dataSource" ref="dataSource"></property>
</bean>
```
	配置完dbAssit后，就可以通过dbAssit来对数据库进行操作了。  

#### 2.3Dao和Service的配置
将dbAssit注入到accountDao中，就可以在持久层利用dbAssit进行数据库操作了。  
```xml
<bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
    <property name="dbAssit" ref="dbAssit"></property>
</bean>
```
同样，将accountDao注入到Service中，因为Service才是逻辑层使用的对象。  
```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
    <!-- 注入dao -->
    <property name="accountDao" ref="accountDao"></property>
</bean>
```
### 3 代码实现
由于Service类才是在逻辑层中被使用的对象，因此，通过反射创建accountService对象，就可以进行CRUD操作了。  
```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml"); IAccountService as = ac.getBean("accountService",IAccountService.class);
```
在每次进行CRUD操作时，使用上述代码创建对象，即可进行CRUD操作。  
此外，还可以使用Java的整合Junit，来自动创建访问数据库的类容器，而不需要手动代码创建。  