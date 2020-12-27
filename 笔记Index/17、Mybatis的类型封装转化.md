## Mybatis 的输出结果封装
### 1数据库列名和实体属性名称不一致
当数据库列名和实体属性名称不一致时，执行查询所有操作，得到的结果如下。  
image1  
只有userName有值的原因是，mysql数据库不区分大小写，因此username可以转为userName，但是可以看到其他名称不同的属性都无法转化,有以下两种解决方法。

### 2 使用别名查询
```html
<select id="findAll" resultMap="userMap">
        <!--select id as userId,username as userName,address as userAddress,sex as userSex,birthday as userBirthday from user;-->
        select * from user;
    </select>
```
比如在使用查询语句的时候将数据库中名称和实体属性名对应起来，解决无法转换的问题。

### 3 配置resultMap
```html
<resultMap id="userMap" type=" com.itheima.domain.User">
        <!-- 主键字段的对应 -->
        <id property="userId" column="id"></id>
        <!--非主键字段的对应-->
        <result property="userName" column="username"></result>
        <result property="userAddress" column="address"></result>
        <result property="userSex" column="sex"></result>
        <result property="userBirthday" column="birthday"></result>
    </resultMap>
```
**建立如上resultMap，建立列名和实体属性名的一一对应关系，将resultType标签替换为resultMap标签，即可通过resultMap将查询结果转化为实体类。**  
- id 标签：用于指定主键字段  
- result 标签：用于指定非主键字段   
- column 属性：用于指定数据库列名   
- property 属性：用于指定实体类属性名称  

### 4 小结
在这一部分，学习了用两种方法来实现数据库结果到实体类的转换，不仅如此，resultMap还可配置一对一、一对多的映射关系。  
