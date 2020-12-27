## 基于注解的IOC
基于注解实现IOC，实际上是将注解写在类的定义中，用各种各样的注解实现IOC。  
### 1 常用注解
#### 1.1 用于创建对象的注解
`@Component`: 把当前类对象存入spring容器中,其属性如下:  
 - value: 用于指定当前类的id. 不写时默认值是当前类名,且首字母改小写  
 
`@Controller`: 将当前表现层对象存入spring容器中  
`@Service`: 将当前业务层对象存入spring容器中  
`@Repository`: 将当前持久层对象存入spring容器中  

这些注解的作用相当于bean.xml中的<bean>标签  
**注意：如果注解中有且只有一个属性要赋值时，且名称是 value，value 在赋值是可以不写**  
`@Controller`,`@Service`,`@Repository`注解的作用和属性与`@Component`是一模一样的,可以相互替代,它们的作用是使三层对象的分别更加清晰.  
#### 1.2 用于数据注入的注解
`@Autowired`: 自动按照成员变量类型注入.  
  
注入过程:  
当spring容器中有且只有一个对象的类型与要注入的类型相同时,注入该对象.  
当spring容器中有多个对象类型与要注入的类型相同时,使用要注入的变量名作为bean的id,在spring 容器查找,找到则注入该对象.找不到则报错.  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/2-3.png)     
**出现位置:** 既可以在变量上,也可以在方法上  
**细节:** 使用注解注入时,set方法可以省略  
`@Qualifier`: 在自动按照类型注入的基础之上,再按照bean的id注入.  
 - 出现位置: 既可以在变量上,也可以在方法上.注入变量时不能独立使用,必须和@Autowire一起使用; 注入方法时可以独立使用.  
 - 属性:value: 指定bean的id  
 
`@Resource`: 直接按照bean的id注入,它可以独立使用.独立使用时相当于同时使用@Autowired和@Qualifier两个注解.  
 - 属性:name: 指定bean的id  
 
`@Value`: 注入基本数据类型和String类型数据  
 - 属性:value: 用于指定数据的值,可以使用el表达式(${表达式})  
#### 1.3 用于改变作用范围的注解
这些注解的作用相当于bean.xml中的<bean>标签的scope属性.  
`@Scope`: 指定bean的作用范围  
 - 属性:value: 用于指定作用范围的取值,"singleton" ,"prototype","request","session","globalsession"    
#### 1.4 生命周期相关的注解
这些注解的作用相当于bean.xml中的<bean>标签的init-method和destroy-method属性  
`@PostConstruct`: 用于指定初始化方法  
  
`@PreDestroy`: 用于指定销毁方法  


### 2 spring的半注解配置
spring的注解配置可以与xml配置并存,也可以只使用注解配置。  
在半注解配置下,spring容器仍然使用`ClassPathXmlApplicationContext`类从xml文件中读取IOC配置,同时在xml文件中告知spring创建容器时要扫描的包.  
使用半注解模式时,上述实例中的beans.xml内容如下:  
```xml
<!-- 告知spring在创建容器时要扫描的包 -->
<context:component-scan 
	base-package="com.itheima"> 
</context:component-scan>
```
	这样一来，所有没有在xml配置文件中配置的类，都可以通过扫描包与定义类时上面的注解来创建容器。  

### 3 spring的纯注解配置
我们发现，现在的xml配置文件中有一句很关键的配置：  
```xml
<!-- 告知spring在创建容器时要扫描的包 -->
<context:component-scan 
	base-package="com.itheima"> 
</context:component-scan>
```
如果这个声明也能用注解配置，那么就离脱离 xml 文件又进了一步。  
因此，在纯注解配置下,可以用配置类替代bean.xml,spring容器使用AnnotationApplicationContext类从spring配置类中读取IOC配置，此时，可以将xml文件彻底移除。  
#### 3.1纯注解配置下的注解
`@Configuration`: 用于指定当前类是一个spring配置类,当创建容器时会从该类上加载注解.获取容器时需要使用AnnotationApplicationContext(有@Configuration注解的类.class).  
`@ComponentScan`: 指定spring在初始化容器时要扫描的包,作用和bean.xml 文件中<context:component-scan base-package="要扫描的包名"/>是一样的.   
其属性如下:  
 - basePackages: 用于指定要扫描的包,是value属性的别名  
`@Bean`: 该注解只能写在方法上,表明使用此方法创建一个对象,并放入spring容器,其属性如下:  
 - name: 指定此方法创建出的bean对象的id  
 
**细节: 使用注解配置方法时,如果方法有参数,Spring框架会到容器中查找有没有可用的bean对象,查找的方式与@Autowired注解时一样的.**  
`@PropertySource`: 用于加载properties配置文件中的配置.例如配置数据源时,可以把连接数据库的信息写到properties配置文件中,就可以使用此注解指定properties配置文件的位置,其属性如下:  
 - value: 用于指定properties文件位置.如果是在类路径下,需要写上"classpath:"  
`@Import`: 用于导入其他配置类.当我们使用@Import注解之后,有@Import注解的类就是父配置类,而导入的都是子配置类. 其属性如下:  
 - value: 用于指定其他配置类的字节码  

#### 3.2纯注解配置实现CRUD
**dao层：**  
```java
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {

    @Autowired	// 自动从spring容器中寻找QueryRunner类型对象注入给runner成员变量
    private QueryRunner runner;		// DBUtil对象,用来执行SQL语句

    public List<Account> findAllAccount() {
		// 功能实现...
    }

    public void saveAccount(Account account) {
        // 功能实现...
    }

    public void deleteAccount(Integer accountId) {
        // 功能实现...
    }
}
```
可以看到，QueryRunner类从xml配置注入变成了注解注入。
**SpringConfiguration类：**  
包config存放配置类，其中`SpringConfiguration`类为主配置类,内容如下:  
```java
@ComponentScan("com.itheima")   //说明此类为配置类
@Import(JdbcConfig.class)        //指定初始化容器时要扫描的包
@PropertySource("classpath:jdbcConfig.properties")   //引入JDBC配置类
public class SpringConfiguration {
}
JDBCConfig类为JDBC配置类,内容如下:
@Configuration									// 说明此类为配置类
@PropertySource("classpath:jdbc.properties")	// 指定配置文件的路径,关键字classpath表示类路径
public class JdbcConfig {

    @Value("${jdbc.driver}")	// 为driver成员属性注入值,使用el表达式
    private String driver;

    @Value("${jdbc.url}")		// 为url成员属性注入值,使用el表达式
    private String url;

    @Value("${jdbc.username}")	// 为usernamer成员属性注入值,使用el表达式
    private String username;

    @Value("${jdbc.password}")	// 为password成员属性注入值,使用el表达式
    private String password;

    // 创建DBUtils对象
    @Bean(name="runner")	// 此将函数返回的bean对象存入spring容器中,其id为runner
    @Scope("prototype")		// 说明此bean对象的作用范围为多例模式,以便于多线程访问
    public QueryRunner createQueryRunner(@Qualifier("ds") DataSource dataSource){
		// 为函数参数datasource注入id为ds的bean对象
        return new QueryRunner(dataSource);
    }

    // 创建数据库连接池对象
    @Bean(name="ds")		// 此将函数返回的bean对象存入spring容器中,其id为ds
    public DataSource createDataSource(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
因此至此，我们可以引入两个配置类`SpringConfiguration`和`JdbcConfig`，再通过`@Import`引入JdbcConfig配置类，再通过`@PropertySource`引入指定jdbc.properties的配置信息，可以将完全摆脱bean.xml了。  
最后，通过`AnnotationApplicationContext`获取数据库连接工具类的容器：  
```java
ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
```
如此一来，就实现了完全使用注解的纯注解方法。  
### 4 基于XML配置与基于注解配置的比较：  
![image](https://github.com/AIchemists/JAVAEE/blob/master/SpringImg/2-4.png)     
基于注解的 spring IoC 配置中，bean 对象的特点和基于 XML 配置是一模一样的。  