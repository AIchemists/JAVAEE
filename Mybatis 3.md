### 本篇是本人看完所有Mybatis教程视频后总结的内容
# Mybatis框架学习3——连接池、事务与sql
主要学习内容：  
- Mybatis的连接池与事务控制
- Mybatis的动态sql语句
- Mybatis的多表查询

## 1、	Mybatis的连接池技术
在 Mybatis 的 SqlMapConfig.xml 配置文件中，通过<dataSource type=”pooled”>来实现 Mybatis 中连接池的配置。  
Mybatis的数据源分为三类：  
- UNPOOLED 不使用连接池的数据源   
- POOLED 使用连接池的数据源  
- JNDI 使用 JNDI 实现的数据源  
  
所谓数据源可以理解为使用连接池技术来更好的管理数据库连接。   

PS:在本次课程学习中，采用的都是POOLED数据源。  
image1  
  
### 1.1	数据源配置
在SqlMapConfig.xml的<environments>中配置数据源  
```
<dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/eesy_mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="1234"/>
            </dataSource>
```
type=”POOLED”,代表mybatis创建pooledDataSource实例，每次使用的时候，从连接池中获取一个连接使用，然后返回使用。
### 1.2数据源获取
配置完数据源之后，mybatis根据工厂模式来创建数据源DataSource实例，并将其放入Configuration对象中的Environment对象中。  

### 1.3Mybatis连接的获取过程
当创建SqlSession对象并执行Sql语句时，Mybatis才会调用datasource对象来创建connection。  
也就是说，**java.sql.Connection对象的创建一直延迟到执行SQL语句的时候。**  
```
InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");   
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);   
SqlSession sqlSession = factory.openSession();   
List list = sqlSession.selectList("findUserById",41);   
System.out.println(list.size());  
```
**在上述代码中，只有当sqlSession执行selectList时候，connection才创建出来。
可以只写属性名）。**  
根据分析PooledDataSource的源代码，可得PooledDataSource得工作原理如下：  
image2  
### 1.4 小结
由此可知，真正打开数据库连接的时间点是代码执行sql语句的时候，也就是说当我们真正要访问数据库的时候才会获取并打开连接，用完了就再立即将数据库连接归还到连接池中，这样很大的节省了连接池资源。
## 2、Mybatis 的事务控制
### 2.1 自动提交事务和手动提交事务
在之前的手动提交事务时，需要在sql的CUD操作执行后调用  
`sqlsession.commit();`
来进行事务的提交，否则对数据库进行的操作无法提交到数据库。  
Mybatis框架的事务控制方式，本身是用 JDBC 的`setAutoCommit()`方法来设置事务提交方式的。  
image3  
在自动提交模式下，所有的Sql语句被执行时作为单个事务提交。  

### 2.2 设置自动提交事务
事实上，需要执行sqlsession.commit();来手动提交事务的原因是我们每次从连接池中取出连接，都会将调用` connection.setAutoCommit(false)`方法，而设置自动提交事务的方法即在创建Sqlsession时，传入一个true作为参数：  
`session = factory.openSession(true);`  
这个参数传入后，会将autoCommit的值设为true并传给连接。  

