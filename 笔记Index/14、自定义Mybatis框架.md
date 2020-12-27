## 自定义Mybatis框架
 在这个部分，将会对Mybatis实现代理模式的原理进行简要的阐述，但由于较为复杂，不会写的太详细。
### 1 工具类 
#### 1.1 Configuration类
该类用于保存连接数据库的用户名、密码、地址等等信息。

#### 1.2 XMLConfigBuilder类
这个类提供一个静态方法 loadConfiguration(),方法的作用是解析主配置文件，把内容填充到DefaultSqlSession类所需要的地方，即通过一些解析技术将SqlMapConfig.xml里配置的驱动、用户名、密码等信息解析获取，获得Configuration类，同时解析持久层接口映射文件（或注解），并封装成map来使用（将接口中的方法名、返回值等信息封装）。

#### 1.3 Executor类
这个类的作用是通过参数获得sql语句的信息以及数据库的连接信息，执行sql操作，获得操作结果并返回。主要是作为执行类。

#### 1.4 DataSourceUtil 类
该类提供一个静态方法getConnection(), 通过getConnection方法将传入的Configuration类对象进行解析，并完成注册驱动和连接操作，并返回这个连接。

#### 1.5 Resources类
Resources提供一个简单的方法，将xml文件读取并得到一个流对象。  


### 2 构建者类
3.2.1 SqlSessionFactoryBuilder类
通过传入SqlMapConfig.xml转化成的输入流，得到数据库连接的配置信息并依次创建一个相应的SqlSessionFactory，这个Factory能够得到一个SqlSession类。

#### 2.2 DefaultSqlSessionFactory 类
该类实现了SqlSessionFactory接口，即实现了openSession方法，可以解析配置信息并以此返回一个SqlSession。

#### 2.3 DefaultSqlSession 类
该类实现了SqlSession接口，即可释放资源，并且能通过传入的解析配置信息，调用DataSourceUtil类的方法在该类中创建持久层连接。同时我们在该类中可以调用Executor类执行查询操作，最重要的是，我们在此类中使用了动态代理，调用了MapperProxyFactory类，增强了持久层所配置的方法。
#### 2.4 MapperProxyFactory类
在这个类中，我们对正在执行的方法进行增强。获取正在执行的方法的所在类名和方法名，从mapper中取出真正要执行的方法，并调用executor来执行需要执行的sql语句。  


### 3 总体流程
image3
在实现这么多类之后，总体上即可完成上一节Mybatis框架所实现的功能，最核心的步骤是：  
**通过创建出的session对象，调用session中的getMapper方法，完成对持久层接口的动态代理，在代理中通过连接和解析出的配置文件中的持久层方法，通过executor类来执行并返回结果。**