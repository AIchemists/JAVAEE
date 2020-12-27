## Mybatis的xml配置实现
### 1传统jdbc
传统jdbc程序执行sql语句的步骤是：
- 加载数据库驱动
`Class.forName("com.mysql.jdbc.Driver");`
- 通过驱动管理类获取数据库连接
 `connection = DriverManager .getConnection("jdbc:mysql://localhost:3306/Mybatis?characterEncoding=utf-8","root", "root");`
- 定义sql语句
 `String sql = "select * from user where username = ?";`
- 获取预处理
 `statement preparedStatement = connection.prepareStatement(sql);`
- 设置sql参数
 `preparedStatement.setString(1, "王五");`
- 向数据库发出sql执行语句
 `resultSet = preparedStatement.executeQuery(); `
- 遍历查询结果集
 `while(resultSet.next()){ System.out.println(resultSet.getString("id")+" "+resultSet.getString("username")); } `
- 释放资源
 `resultSet.close(); preparedStatement.close(); connection.close();` 
 
可以看出，传统的jdbc方法有很多问题：  
1、数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。   
2、Sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 java 代码。   
3、使用 preparedStatement 向占有位符号传参数存在硬编码，因为 sql 语句的 where 条件不一定，可能 多也可能少，修改 sql 还要修改代码，系统不易维护。  
4、对结果集解析存在硬编码（查询列名），sql 变化导致解析代码变化，系统不易维护，如果能将数据库记 录封装成 pojo 对象解析比较方便。

### 2 Mybatis框架配置实现
#### 2.1 持久层接口与映射文件
在下载了Mybatis的jar包并添加到pom.xml后，编写持久层的接口  
```java
public interface IUserDao {
    List<User> findAll();
    }
```
这是一个查询所有用户的操作。  
对应这个持久层接口，同时创建这个接口的映射文件  
```html
<mapper namespace="com.itheima.dao.IUserDao">
    <!--配置查询所有-->
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>
```
id为方法名称，resultType为返回类型的全限定类名。  
其中，namespace的值即为持久层接口的位置。  
注意，该映射文件必须和持久层接口在相同的包中，即二者的路径必须相同。其次，必须以持久层接口名称命名文件名，扩展名是.xml。  
image 1-1
#### 2.2 编写SqlMapConfig.xml配置文件
``` html
<!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/eesy_Mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="1234"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <mappers>
        <mapper resource="com/itheima/dao/IUserDao.xml"/>
</mappers>
```
注意，该处的<mappers>中resource代表使用xml文件映射的方法来实现持久层接口，还有两种写法(class和package)将会在之后阐述。
 
#### 2.3 初步实现Mybatis
 ```java
        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
        //4.使用SqlSession创建Dao接口的代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
        //6.释放资源
        session.close();
        in.close();
 ```
  其中，SqlSessionFactory和SqlSession即为Mybatis给我们提供的编写的持久层类。SqlSessionFactory读取了SqlMapConfig.xml中的配置信息，产生真正操作数据库的SqlSession,而SqlSession则用来帮开发者生成持久层接口的代理对象，解析接口的映射文件，通过代理方式执行CRUD操作。  
### 3 小结
  在这一部分，可以对比传统jdbc和Mybatis框架实现持久层操作的过程，显然Mybatis远比jdbc方便。在这个阶段，我们只用Mybatis创建了持久层接口的映射文件与sql数据库的配置文件，再使用Mybatis提供的持久层类来实现自动代理，即可完成持久层操作。  
  image2  
  