## Mybatis的DAO传统开发方式
在这个部分，将会学习基于传统编写 Dao 实现类的开发方式。但是由于是了解部分内容，不会写的太详细。  
### 1 IUserDao接口
这个接口即普通的持久层接口，包含了普通的CRUD方法。

### 2 UserDaoImpl类
这个类实现了IUserDao接口，有一个SqlSessionFactory类的对象，在每个接口的方法实现中，通过factory创建一个SqlSession,直接通过session的select、insert、update、delete等等方法来完成CRUD操作，结束后再关闭session。

### 3 执行测试
在执行时，直接通过相同的方法获得SqlSessionfactory，并将这个factory传入UserDaoImpl类创建一个Dao接口的实现类，再用这个实现类来完成CRUD操作即可。
```java
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = factory.openSession();
        //4.获取dao的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
```