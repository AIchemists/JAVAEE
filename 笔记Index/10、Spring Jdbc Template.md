## Spring JDBC Template
### 1	JDBC Template是什么  
JdbcTemplate是Spring框架中提供的一个对象,对原始的JDBC API进行简单封装,其用法与DBUtils类似。JdbcTemplate是Spring对JDBC的封装，目的是使JDBC更加易于使用。JdbcTemplate是Spring的一部分。JdbcTemplate处理了资源的建立和释放。他帮助我们避免一些常见的错误，比如忘了总要关闭连接。他运行核心的JDBC工作流，如Statement的建立和执行，而我们只需要提供SQL语句和提取结果。  
### 2	JDBCTemplate创建
JdbcTemplate对象在执行sql语句时也需要一个数据源，可以选择三种数据源：C3P0或者DBCP,也可以使用Spring的内置数据源`DriverManagerDataSource`。  
然后在bean.xml中配置三种数据源即可。  
接着就可以创建JdbcTemplate对象，向JdbcTemplate中注入数据源创建对象,在bean.xml中配置如下:  
```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```
### 3JdbcTemplate的增删改查操作
#### 3.1增删改操作
与DBUtils十分类似,JdbcTemplate的增删改操作使用其update("SQL语句", 参数...)方法。  
创建JDBC Template:  
```java
    //1.获取Spring容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
    //2.根据id获取JdbcTemplate对象
    JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
    //3.执行增加操作
    jt.update("insert into account(name,money)values(?,?)","名字",5000);}
    //4.执行删除操作
    jt.update("delete from account where id = ?",6);
    //5.执行更新操作
    jt.update("update account set money = money-? where id = ?",300,6);
```
#### 3.2实现查询
与DBUtils十分类似,JdbcTemplate的查询操作使用其query()方法,其参数如下:  
 - `String sql`: SQL语句  
 - `RowMapper<T> rowMapper`: 指定如何将查询结果ResultSet对象转换为T对象  
 - `@Nullable Object... args`: SQL语句的参数  
 
其中`RowMapper`类类似于DBUtils的`ResultSetHandler`类,可以自己写一个实现类,但常用Spring框架内置的实现类`BeanPropertyRowMapper<T>(T.class)`  

查询所有:  
```java
    List<Account> accounts = jdbcTemplate.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
```
查询一条记录:  
```java
    List<Account> accounts = jdbcTemplate.query("select * from account where id = ?", new BeanPropertyRowMapper<Account>(Account.class), accountId);
```
聚合查询:  

JdbcTemplate中执行聚合查询的方法为`queryForObject()`方法,其参数如下:  			
 - `String sql`: SQL语句  
 - `Class<T> requiredType`: 返回类型的字节码  
 - `@Nullable Object... args`: SQL语句的参数  
 
```java
    //3.聚合查询
    Integer total = jt.queryForObject("select count(*) from account where money > ?", Integer.class, 500);
```
### 4在DAO层使用JdbcTemplate
#### 4.1直接使用JdbcTemplate
在DAO层使用JdbcTemplate,需要给DAO注入JdbcTemplate对象。  
```java
public class AccountDaoImpl implements IAccountDao {   
    private JdbcTemplate jdbcTemplate;	// JdbcTemplate对象
    // JdbcTemplate对象的set方法
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
  // 其它DAO层方法...
}
```
产生的问题:  
当dao 有很多时，每个 dao 都有一些重复性的代码，都需要注入JDBCTemplate并且都要写set方法。  
用下面一种方法可以解决这个问题。  
#### 4.2 DAO层对象继承JdbcDaoSupport类
在实际项目中,我们会创建许多DAO对象,若每个DAO对象都注入一个JdbcTemplate对象,会造成代码冗余。  
 - 实际的项目中我们可以让DAO对象继承Spring内置的`JdbcDaoSupport`类。在`JdbcDaoSupport`类中定义了JdbcTemplate和DataSource成员属性,在实际编程中,只需要向其注入DataSource成员即可,DataSource的set方法中会注入JdbcTemplate对象。
 - DAO的实现类中调用父类的`getJdbcTemplate()`方法获得JdbcTemplate()对象。  
`JdbcDaoSupport`类的源代码如下:  
```java
public abstract class JdbcDaoSupport extends DaoSupport {
    
    @Nullable
    private JdbcTemplate jdbcTemplate;	// 定义JdbcTemplate成员变量
    public JdbcDaoSupport() {
    }
    // DataSource的set方法,注入DataSource时调用createJdbcTemplate方法注入JdbcTemplate
    public final void setDataSource(DataSource dataSource) {
        if (this.jdbcTemplate == null || dataSource != this.jdbcTemplate.getDataSource()) {
            this.jdbcTemplate = this.createJdbcTemplate(dataSource);
            this.initTemplateConfig();
        }
    }
    // 创建JdbcTemplate,用来被setDataSource()调用注入JdbcTemplate
    protected JdbcTemplate createJdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    // JdbcTemplate的get方法,子类通过该方法获得JdbcTemplate对象
    @Nullable
    public final JdbcTemplate getJdbcTemplate() {
        return this.jdbcTemplate;
    }
    @Nullable
    public final DataSource getDataSource() {
        return this.jdbcTemplate != null ? this.jdbcTemplate.getDataSource() : null;
    }
    public final void setJdbcTemplate(@Nullable JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        this.initTemplateConfig();
    }
    // ...
}
```
在bean.xml中,我们只要为DAO对象注入DataSource对象即可,让JdbcDaoSupport自动调用JdbcTemplate的set方法注入JdbcTemplate  
```xml
<bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <!--不用我们手动配置JdbcTemplate成员了-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```
在实现DAO接口时,我们通过`super.getJdbcTemplate()`方法获得JdbcTemplate对象。  
```java
@Override
    public Account findAccountById(Integer id) {
		//调用继承自父类的getJdbcTemplate()方法获得JdbcTemplate对象
        List<Account> list = getJdbcTemplate().query("select * from account whereid = ? ", new AccountRowMapper(), id);
        return list.isEmpty() ? null : list.get(0);
    }
```
注意：这种配置方法存在一个问题: 因为我们不能修改`JdbcDaoSupport`类的源代码,DAO层的配置就只能基于xml配置,而不再可以基于注解配置了。  
**两种方法的区别：**  
**第一种在 Dao 类中定义 JdbcTemplate 的方式，适用于所有配置方式（xml 和注解都可以）。**   
**第二种让 Dao 继承 JdbcDaoSupport 的方式，只能用于基于 XML 的方式，注解用不了。**  

### 5、小结
在这一部分学习内容中，主要的内容是了解JDBCTemplate，了解JDBCTemplate是什么以及怎样使用JDBCTemplate。并且有两种方法来使用JDBCTemplate：Dao层直接使用和Dao层对象继承JdbcDaoSupport类。两种方法各有优缺点。