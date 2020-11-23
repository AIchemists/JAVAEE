### 本篇是本人看完所有Mybatis教程视频后总结的内容
# Mybatis框架学习4——缓存与延迟加载
主要学习内容：
- Mybatis的延迟加载
- Mybatis的两级缓存机制
- Mybatis注解实现复杂的配置

## 1、	Mybatis的延迟加载
在上一部分中，学习了Mybatis的一对一、一对多及多对多关系的关联查询，但是在实际开发过程中很多时候我们并不需要总是在加载用户信息时就一定要加载他的账户信息，因此在开发过程中就可以使用Mybatis提供的延迟加载。
延迟加载就是在需要用到数据时才进行加载，不需要用到数据时就不加载数据。延迟加载也称懒加载。  
**因此采用延迟加载时，先从单表查询，需要时再从关联表去关联查询，大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。**

### 1.1	延迟加载的设置
当我们查询账户和相关的用户时，使用<association>标签来完成关联查询，配置的resultMap如下:  
```html
<resultMap id="accountUserMap" type="account">
        <id property="id" column="id"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <association property="user" column="uid" javaType="user" select="com.itheima.dao.IUserDao.findById"></association>
    </resultMap>
```
其中，
- select： 填写我们要调用的 select 映射的 id  
- column ： 填写我们要传递给 select 映射的参数  

 
另外需要的操作就是在SqlMapConfig.xml文件中添加开启延迟加载的配置。  
```html
    <settings>
        <!--开启Mybatis支持延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"></setting>
    </settings>
    ```
在配置文件中配置了如上信息后，Mybatis的延迟加载就开启了。  
其中，aggressiveLazyLoading为false表示在进行关联查询后，每使用一个属性就加载哪个属性。  
测试：当我们使用关联语句查询账户时，可以看到测试结果：  
image1
