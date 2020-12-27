## 	Mybatis的连接池技术
在 Mybatis 的 SqlMapConfig.xml 配置文件中，通过<dataSource type=”pooled”>来实现 Mybatis 中连接池的配置。  
Mybatis的数据源分为三类：  
- UNPOOLED 不使用连接池的数据源   
- POOLED 使用连接池的数据源  
- JNDI 使用 JNDI 实现的数据源  
  
所谓数据源可以理解为使用连接池技术来更好的管理数据库连接。   

PS:在本次课程学习中，采用的都是POOLED数据源。  
image1  
  
### 1	数据源配置
在SqlMapConfig.xml的<environments>中配置数据源  
```html
<dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/eesy_mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="1234"/>
            </dataSource>
```
type=”POOLED”,代表mybatis创建pooledDataSource实例，每次使用的时候，从连接池中获取一个连接使用，然后返回使用。
### 2数据源获取
配置完数据源之后，mybatis根据工厂模式来创建数据源DataSource实例，并将其放入Configuration对象中的Environment对象中。  

### 3Mybatis连接的获取过程
当创建SqlSession对象并执行Sql语句时，Mybatis才会调用datasource对象来创建connection。  
也就是说，**java.sql.Connection对象的创建一直延迟到执行SQL语句的时候。**  
```java
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
### 4 小结
由此可知，真正打开数据库连接的时间点是代码执行sql语句的时候，也就是说当我们真正要访问数据库的时候才会获取并打开连接，用完了就再立即将数据库连接归还到连接池中，这样很大的节省了连接池资源。