## 	基于代理dao实现CRUD
在第一部分中只用了一个查询所有用户的例子，在这个部分中会详细的学习其他CRUD的实现方法。

### 1 通过id查询用户（sql语句需要传入基本类型参数）
当通过id查询用户时，需要传入int类型的用户id，映射文件如下  
 ```html
 <select id="findById" parameterType="INT" resultType=" "com.itheima.domain.User" ">
        select * from user where id = #{uid}
 </select>
 ```
新加了parameterType属性表示参数类型，sql中的#{ }表示占位符，执行时被替换为实际的数据。  

### 2 保存新用户（ sql语句需要传入类的对象）
当执行插入一个新用户操作时，需要传入一个user对象。  
```html
<insert id="saveUser" parameterType="com.itheima.domain.User">
insert into user(username,address,sex,birthday)values(#{userName},#{userAddress},#{userSex},#{userBirthday});
    </insert>
 ```
此时传入User类，因此parameterType中写全限定类名。而#{ }中的名词要写User对象中的属性名称（因为在parameterType指定了类名，因此可以只写属性名）。  
**注意：在对数据库实现增删改时，需要使用session.commit()来实现对事务的提交，下同。**
```html
<selectKey keyProperty="userId" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
 ```
另外，可以添加该配置来获取新增用户的id，使实体用户id同步。  

### 3用户更新
```html
<update id="updateUser" parameterType="USER">
        update user set username=#{userName},address=#{userAddress},sex=#{userAex},birthday=#{userBirthday} where id=#{userId}
    </update>
```
使用<update>标签，没太大区别。

### 4 用户删除
```html
<delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id = #{uid}
    </delete>
 ```
使用<delete>标签，没太大区别。

### 5 模糊查询
①sql语句占位符为#{ }时，查询方法传入参数时就需要给定模糊查询的标识%。即  
```java
List users = userDao.findByName("%王%");
```
②当占位符为`’%${value}%’`时（写法固定），此时传入的参数不需要表示%。  
```java
List users = userDao.findByName("王");
```  
#{}表示一个占位符号 通过#{}可以实现 preparedStatement 向占位符中设置值，自动进行 java 类型和 jdbc 类型转换，**注意：#{}可以有效防止 sql 注入**。#{}可以接收简单类型值或 pojo 属性值。 如果 parameterType 传输单个简单类 型值，#{}括号中可以是 value 或其它名称。   
${}表示拼接 sql 串 通过${}可以将 parameterType 传入的内容拼接在 sql 中且不进行 jdbc 类型转换， ${}可以接收简 单类型值或 pojo 属性值，**如果 parameterType 传输单个简单类型值，则${}括号中只能是 value。**
### 6 pojo包装对象
若将user作为pojo类的一个属性，在传参的时候用pojo类，则sql语句中的参数带上user类名即可。  
```html
<select id="findUserByVo" parameterType="com.itheima.domain.QueryVo" resultType=" "com.itheima.domain.User" ">
        select * from user where username like #{user.username}
    </select>
```
### 7 小结
Mybatis与Jdbc相比，首先mybatis可通过在SqlMapConfig.xml中配置数据链接池来管理数据库连接（后面会学习），其次，mybatis将sql语句写在配置文件中，使之与代码分离。而且，mybatis自动将java对象映射至sql语句，并自动将sql执行结果映射至java对象。