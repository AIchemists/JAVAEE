## Spring的事务控制
事务是指数据库中的一组逻辑操作，这个操作的特点就是在该组逻辑中，所有的操作要么全部成功，要么全部失败。在各个数据具有特别紧密的联系时，最好是使用数据库的事务来完成逻辑处理。  
1、JavaEE 体系进行分层开发,事务处理位于业务层,Spring提供了分层设计业务层的事务处理解决方案  
2、Spring 框架为我们提供了一组事务控制的接口,这组接口在spring-tx-5.0.2.RELEASE.jar中  
3、Spring 的事务控制都是基于AOP的,它既可以使用配置的方式实现,也可以使用编程的方式实现.推荐使用配置方式实现  
### 1 Spring中事务控制的API
#### 1.1 PlatformTransactionManager
**PlatformTransactionManager**接口是Spring提供的事务管理器,它提供了操作事务的方法如下:  
1.TransactionStatus getTransaction(TransactionDefinition definition): 获得事务状态信息  
2.void commit(TransactionStatus status): 提交事务  
3.void rollback(TransactionStatus status): 回滚事务  
在实际开发中我们使用其实现类:  
1.org.springframework.jdbc.datasource.DataSourceTransactionManager使用SpringJDBC或iBatis进行持久化数据时使用  
2.org.springframework.orm.hibernate5.HibernateTransactionManager使用Hibernate版本进行持久化数据时使用  
#### 1.2 TransactionDefinition
**TransactionDefinition:** 事务定义信息对象,提供查询事务定义的方法如下:    
1.String getName(): 获取事务对象名称  
2.int getIsolationLevel(): 获取事务隔离级别,设置两个事务之间的数据可见性  
事务的隔离级别由弱到强,依次有如下五种:  
 - `ISOLATION_DEFAULT`: Spring事务管理的的默认级别,使用数据库默认的事务隔离级别.  
 - `ISOLATION_READ_UNCOMMITTED`: 读未提交.  
 - `ISOLATION_READ_COMMITTED`: 读已提交.  
 - `ISOLATION_REPEATABLE_READ`: 可重复读.  
 - `ISOLATION_SERIALIZABLE`: 串行化.  
 
3.getPropagationBehavior(): 获取事务传播行为,设置新事务是否事务以及是否使用当前事务.  
我们通常使用的是前两种: REQUIRED和SUPPORTS.事务传播行为如下:  
 - `REQUIRED`: Spring默认事务传播行为. 若当前没有事务,就新建一个事务;若当前已经存在一个事务中,加入到这个事务中.增删改查操作均可用  
 - `SUPPORTS`: 若当前没有事务,就不使用事务;若当前已经存在一个事务中,加入到这个事务中.查询操作可用  
 - `MANDATORY`: 使用当前的事务,若当前没有事务,就抛出异常  
 - `REQUERS_NEW`: 新建事务,若当前在事务中,把当前事务挂起  
 - `NOT_SUPPORTED`: 以非事务方式执行操作,若当前存在事务,就把当前事务挂起  
 - `NEVER`:以非事务方式运行,若当前存在事务,抛出异常  
 - `NESTED`:若当前存在事务,则在嵌套事务内执行;若当前没有事务,则执行REQUIRED类似的操作  
 
4.	int getTimeout(): 获取事务超时时间. Spring默认设置事务的超时时间为-1,表示永不超时  
5.	boolean isReadOnly(): 获取事务是否只读. Spring默认设置为false,建议查询操作中设置为true  
#### 1.3 TransactionStatus
**TransactionStatus:** 事务状态信息对象,提供操作事务状态的方法如下:  
1.	void flush(): 刷新事务  
2.	boolean hasSavepoint(): 查询是否存在存储点  
3.	boolean isCompleted(): 查询事务是否完成  
4.	boolean isNewTransaction(): 查询是否是新事务  
5.	boolean isRollbackOnly(): 查询事务是否回滚  
6.	void setRollbackOnly(): 设置事务回滚  

### 2 Spring通过xml实现事务控制
#### 2.1 准备工作
首先，导入jar包到项目的lib目录，接着创建bean.xml并导入约束，准备实体类Account和持久层类`AccountDaoImpl`、`IAccountDao`、逻辑层类`AccountServiceImpl`、`IAccountService`。  
同时，在bean.xml中配置数据源以及要扫描的包。  
在这里，使用JDBCTemplate作为数据库的连接操作工具。  
```xml
<!--配置 创建Spring容器时要扫描的包-->
<context:component-scan base-package="com.itheima"></context:component-scan>
<!--配置 数据源-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/spring_day02"></property>
    <property name="username" value="root"></property>
    <property name="password" value="1234"></property>
</bean>    
<!--配置 JdbcTemplate-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```
#### 2.2 配置xml
Spring的配置式事务控制本质上是基于AOP的,因此下面我们在bean.xml中配置事务管理器`PlatformTransactionManager`对象并将其与AOP配置建立联系。  
1、	配置事务管理器并注入数据源  
```xml
<!--向Spring容器中注入一个事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--注入数据源-->
    <property name="dataSource" ref="dataSource" />
</bean>
```
2、	配置事务通知并在通知中配置其属性  
使用<tx:advice>标签声明事务配置,其属性如下:  
 - id: 事务配置的id  
 - transaction-manager: 该配置对应的事务管理器  
 
<tx:advice>标签内包含<tx:attributes>标签表示配置事务属性  
<tx:attributes>标签内包含<tx:method>标签,为切面上的方法配置事务属性  
<tx:method>标签的属性如下:  
 - name: 拦截到的方法,可以使用通配符*  
 - isolation: 事务的隔离级别,Spring默认使用数据库的事务隔离级别  
 - propagation: 事务的传播行为,默认为REQUIRED.增删改方法应该用REQUIRED,查询方法可以使用SUPPORTS  
 - read-only: 事务是否为只读事务,默认为false.增删改方法应该用false,查询方法可以使用true  
 - timeout: 指定超时时间,默认值为-1,表示永不超时  
 - rollback-for: 指定一个异常,当发生该异常时,事务回滚;发生其他异常时,事务不回滚.无默认值,表示发生任何异常都回滚  
 - no-rollback-for: 指定一个异常,当发生该异常时,事务不回滚;发生其他异常时,事务回滚.无默认值,表示发生任何异常都回滚  
 
 ```xml
<!-- 配置事务的通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--匹配所有方法-->
        <tx:method name="*" propagation="REQUIRED" read-only="false" rollback-for="" no-rollback-for=""/>
        
        <!--匹配所有查询方法-->
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
</tx:advice>
```
**注意：第二个<tx:method>匹配得更精确,所以对所有查询方法,匹配第二个事务管理配置;对其他查询方法,匹配第一个事务管理配置**  
3、配置AOP并为事务管理器事务管理器指定切入点  
```xml
<!--配置AOP-->
<aop:config>
    <!-- 配置切入点表达式-->
    <aop:pointcut id="pt1" expression="execution(* cn,maoritian.service.impl.*.*(..))"></aop:pointcut>
    <!--为事务通知指定切入点表达式-->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
</aop:config>
```
### 3 Spring通过注解实现事务控制
spring中基于注解 的声明式事务控制配置步骤：  
1.	配置事务管理器  
2.	开启spring对注解事务的支持  
3.	在需要事务支持的地方使用@Transactional注解  
首先需要在bean.xml中配置事务管理器:  
```xml
<!--向Spring容器中注入一个事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--注入数据源-->
    <property name="dataSource" ref="dataSource"></property>
</bean>
```
接着需要开启spring对注解事务的支持:  
```xml
<!-- 开启spring对注解事务的支持-->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```
#### 3.1半注解方式
在业务层使用`@Transactional`注解,其参数与<tx:method>的属性一致。  
该注解可以加在接口,类或方法上  
1.	对接口加上`@Transactional`注解,表示对该接口的所有实现类进行事务控制  
2.	对类加上`@Transactional`注解,表示对类中的所有方法进行事务控制  
3.	对具体某一方法加以`@Transactional`注解,表示对具体方法进行事务控制  
三个位置上的注解优先级依次升高。  
```java
// 业务层实现类,事务控制应在此层
@Service("accountService")
@Transactional(propagation = Propagation.REQUIRED, readOnly = false)    // 读写型事务配置
public class AccountServiceImpl implements IAccountService {

    @Autowired
    private IAccountDao accountDao;

    @Override
    @Transactional(propagation = Propagation.SUPPORTS, readOnly = true) // 只读型事务配置,会覆盖上面对类的读写型事务配置 
    public Account findAccountById(Integer accountId) {
        return accountDao.findAccountById(accountId);
    }
//…其他操作 
}
```
这里，配置在方法上的注解会覆盖前面的其他注解。

#### 3.2纯注解方式
如果不使用xml配置事务,就要在cn.itheima.config包下新建一个事务管理配置类`TransactionConfig`,对其加上`@EnableTransactionManagement`注解以开启事务控制。  
事务控制配置类`TransactionConfig`类：  
```java
@Configuration                  
@EnableTransactionManagement    // 开启事务控制
public class TransactionConfig {
    // 创建事务管理器对象
    @Bean(name="transactionManager")
    public PlatformTransactionManager createTransactionManager(@Autowired DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```
JDBC配置类`JdbcConfig`类:
```java
@Configuration                                  
@PropertySource("classpath:jdbcConfig.properties")  
public class JdbcConfig {

    @Value("${jdbc.driver}")    
    private String driver;

    @Value("${jdbc.url}")   
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")  
    private String password;

    // 创建JdbcTemplate对象
    @Bean(name="jdbcTemplate")
    @Scope("prototype") 
    public JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
    
    // 创建数据源对象
    @Bean(name="dataSource")
    public DataSource createDataSource(){
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}
```
Spring主配置类`SpringConfig`:
```java
@Configuration
@ComponentScan("cn.itheima")
@Import({JdbcConfig.class, TransactionConfig.class})
public class SpringConfig {
}
```
### 4 小结
本部分了解了Spring中事务控制的API，同时还学习了使用Spring事务控制的步骤和方法。在Spring事务控制的实现中，也可以使用xml配置文件或者注解方式实现，可以使用半注解方式与纯注解方式实现。  