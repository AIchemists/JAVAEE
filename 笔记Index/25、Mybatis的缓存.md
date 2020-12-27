## Mybatis 缓存
### 1 Mybatis一级缓存
一级缓存是 SqlSession 级别的缓存，只要 SqlSession 没有 flush 或 close，它就存在。  
mybatis的一级缓存不需要手动打开，而是自身存在一级缓存。  
#### 1.1一级缓存的测试
首先，使用查找用户方法来测试Mybatis的一级缓存。  
```html
<select id="findById" parameterType="INT" resultType="user" useCache="true">
        select * from user where id = #{uid}
    </select>
```
在测试文件中编写：  
```java
User user1 = userDao.findById(41);
        System.out.println(user1);
        User user2 = userDao.findById(41);
        System.out.println(user2);
        System.out.println(user1 == user2);
```
在测试中我们对id为41的用户进行了两次查询，结果如下：  
 image3  
发现，虽然在上面的代码中查询了两次，但最后只执行了一次数据库操作，这就是 Mybatis 提供的一级缓存在起作用。因为一级缓存的存在，导致第二次查询 id 为 41 的记录时，并没有发出sql语句从数据库中查询数据，而是从同一个session的一级缓存中查询，因此两次查询出来的User是同一个User。  
#### 1.2一级缓存的清除
如果sqlSession执行了查询操作之后，执行commit操作（执行插入、更新、删除），**则Mybatis会清空 SqlSession 中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。**   

### 2 Mybatis二级缓存
与一级缓存不同的是，Mybatis的二级缓存是存在于sessionfactory中的。二级缓存是 mapper 映射级别的缓存，多个 SqlSession 去操作同一个 Mapper 映射的 sql 语句，多个 SqlSession 可以共用二级缓存，二级缓存是跨 SqlSession 的。  
image4  
#### 2.1 二级缓存的设置
与一级缓存不同的是，二级缓存需要手动开启，即Mybatis默认是关闭二级缓存的。  
首先在SqlMapConfig.xml中开启二级缓存：  
```html
<settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
```
第二步，在相关持久层接口的Mapper映射文件中加上二级缓存，即加上<cache/>标签。  
第三步，在statement上添加上useCache=”true”这个属性值，表示这个statement要使用二级缓存。**注意：针对每次查询都需要最新的数据 sql，要设置成 useCache=false，禁用二级缓存。**  

#### 2.2二级缓存的测试
在测试中使用如下代码  
```java
SqlSession sqlSession1 = factory.openSession();
        IUserDao dao1 = sqlSession1.getMapper(IUserDao.class);
        User user1 = dao1.findById(41);
        System.out.println(user1);
        sqlSession1.close();//一级缓存消失
        SqlSession sqlSession2 = factory.openSession();
        IUserDao dao2 = sqlSession2.getMapper(IUserDao.class);
        User user2 = dao2.findById(41);
        System.out.println(user2);
        sqlSession2.close();
        System.out.println(user1 == user2);
```
当我们分别用同一个factory的两个sqlSession来进行查询的时候，我们发现用第二个sqlSession语句查询并没有在数据库中使用sql查询语句，说明第二次的查询使用了二级缓存。**但是需要注意的是，两次查询出来的user1和user2并不是同一个，因为存在factory中的二级缓存是数据而不是类，因此两个并不是同一个类。**  
因此我们要注意，当我们在使用二级缓存时，所缓存的类一定要实现 java.io.Serializable 接口，这种就可以使用序列化 方式来保存对象。  

### 3小结
Mybatis给我们提供了两级的缓存。mybatis一级缓存存在sqlsession中，存的是对象，不用开启。二级缓存存在sessionFactory中，存的是数据，需要手动开启。当对数据库执行了增删改操作时，缓存的内容会消失。  