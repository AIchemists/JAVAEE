# Mybatis
## 代理DAO
### 自定义类
最终工具类Executor, Executor需要mapper和connection，其中mapper记录查询语句和返回数据类型，connection创建数据库连接。在代理对象MapperProxy中增强调用Executor，增强方法是获取mapper，mapper的获取方法是由工具类XMLConfigBuilder来读取配置文件，并存入Configuration对象中。  
总的来说就两件事：查询所有和创建代理对象调用查询所有操作。  
![image](https://github.com/AIchemists/JAVAEE/blob/master/MybatisImage/1.png)
![image](https://github.com/AIchemists/JAVAEE/blob/master/MybatisImage/2.png)
![image](https://github.com/AIchemists/JAVAEE/blob/master/MybatisImage/3.png)
用注解方式就是获取映射mapper的方法改为通过注解获取方法。  
## 表的列名与实体的属性名的对应关系
windows下的mysql数据库表中的属性名和实体的属性名不区分大小写，但是名称不同的无法自动转换，需要在sql语句中取别名（执行效率）或用ResultMap配置（开发效率）对应关系。  
## 不使用代理DAO，自己写实现类
即直接使用Session.Selectlist()等方法。（代理Dao本质上也是用Selectlist，只不过Mybatis帮我们写了Dao类）  
Session的CURD方法，本质上是Session类—DefaultSqlSession类-Executor类-CachingExecutor-Delegate.Update-BaseExecutor-SimpleExecutor-HandlerUpdate-PrepareedStatementHandler执行CURD操作。   
## Mybatis缓存机制
Mybatis一级缓存存在SqlSession中，存的是对象，不用开启。  
二级缓存存在SessionFactory中，存的是数据，需要手动开启。  
## 关于SqlMapConfig.xml
<Mappers>-<packages>接口所在位置，用于指定Dao接口所在的包  
<TypeAliases>配置别名，实体类名就是别名  
  
## 最简单的注解方法
CURD4个注解，不用映射文件接口.xml文件也行，两个都有会报错，不管mapper中用package还是class，只要有接口.xml映射文件就会报错。所以一个Dao要么用注解，要么用xml。通常来说所有Dao都统一用一个。  
用注解方法，读取配置文件能实现的缓存、多对多、一对一映射功能都可以实现。
  
