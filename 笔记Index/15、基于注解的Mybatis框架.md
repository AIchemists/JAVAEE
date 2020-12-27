## 基于注解的Mybatis框架 
若为基于注解的Mybatis类，则不用创建持久层接口的配置文件来解析函数，而用注解的方法来获取持久层接口中的函数。  
### 1 定义@Select注解
```java
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD) 
public @interface Select
 { String value(); }
```
### 2 注释持久层接口
```java
public interface IUserDao {
    @Select("select * from user")
    List<User> findAll();
}
```
在解析dao接口获取mapper的时候，方法名直接获取，返回值则是方法名前的类型,可以解析出来。  
 
### 3 修改SqlMapConfig.xml
``` html
<mappers>
        <mapper class="com.itheima.dao.IUserDao"/>
    </mappers>
 ```
在<mappers>中用class来指定被注解的dao全限定类名

### 4 注解实现小结
如此即可完成注解实现Mybatis框架，可以对比配置方法和注解方法，发现注解方法可以通过对Dao接口的注释来替代接口的映射文件，满足方法的解析与获取操作。