## Mybatis的多表查询
视频中所使用的案例为为简单的用户和账户的模型来分析 Mybatis 多表关系。用户为 User 表，账户为Account 表。一个用户（User）可以有多个账户（Account）。  
具体关系如下：  
  iamge4
  
  ### 1 一对一查询
因为一个账户信息只能供某个用户使用，所以从查询账户信息出发关联查询用户信息为一对一查询。有两种方式实现。
#### 1.1定义AccountUser类
这个类包含了账户信息和用户信息，这样就可以接受查询结果返回的账户信息和用户信息了。  
```html
<select id="findAllAccount" resultType="accountuser">
        select a.*,u.username,u.address from account a , user u where u.id = a.uid;
    </select>
```
在持久层接口的映射文件中将resultType设为accountUser，使用sql的表关联查询，这样就同时接收了账户信息和用户信息。  
#### 1.2 定义resultMap
在Account类中添加User类作为Account类的一个属性。  
这样的话，查询结果只用返回Account类，因为 Account 类中包含了一个 User 类的对象，它可以封装账户所对应的用户信息。  
```html
<resultMap id="accountUserMap" type="account">
        <id property="id" column="aid"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <association property="user" column="uid" javaType="user">
            <id property="id" column="id"></id>
            <result column="username" property="username"></result>
            <result column="address" property="address"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
        </association>
    </resultMap>
```

使用<resultMap>标签来建立对应关系，**而<association>标签表示一对一的关系映射**，用于配置User类中的信息。  
  
### 2 一对多查询
如果从用户信息出发查询用户下的账户信息则为一对多查询，因为一个用户可以有多个账户。  
首先，sql语句同样使用关联表查询，在这里使用左外连接查询比较合适。  
在一对多的查询中，我们使用resultMap的方式来实现，因此，在User类中类加入 List<Account>来获取所有账户信息。  
```java
private List<Account> accounts;
  ```
在Dao映射文件中，配置resultMap:  
  ```html
<resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <collection property="accounts" ofType="account">
            <id column="aid" property="id"></id>
            <result column="uid" property="uid"></result>
            <result column="money" property="money"></result>
        </collection>
    </resultMap>
  ```
  
**其中<collection>是用于建立一对多中集合属性的对应关系** ,<property>指定关联查询的结果集存储在 User 对象的上哪个属性，<ofType>用于指定集合元素的数据类型。   
  

### 3 多对多查询
在前两节中，使用 Mybatis 实现一对多关系的维护。**多对多关系其实我们看成是双向的一对多关系**。  
查询角色我们需要用到Role表，但角色分配的用户的信息我们并不能直接找到用户信息，而是要通过中 间表(USER_ROLE 表)才能关联到用户信息，从 User 出发，我们也可以发现一个用户可以具有多个角色，这样用户到角色的关系也还是一对多关系。这样 我们就可以认为 User 与 Role 的多对多关系，可以被拆解成两个一对多关系来实现。因此，我们使用的sql语句同样要通过连接表查询。  
**因此，只用分别在Role和UserDao接口的映射文件中配置resultMap并使用一对多的<collection>标签，就可以完成多对多查询。**   
代码如下：
```html
<!--定义user表的ResultMap-->
<resultMap id="userMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <collection property="roles" ofType="role">
            <id property="roleId" column="rid"></id>
            <result property="roleName" column="role_name"></result>
            <result property="roleDesc" column="role_desc"></result>
        </collection>
    </resultMap>

<!--定义role表的ResultMap-->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="rid"></id>
        <result property="roleName" column="role_name"></result>
        <result property="roleDesc" column="role_desc"></result>
        <collection property="users" ofType="user">
            <id column="id" property="id"></id>
            <result column="username" property="username"></result>
            <result column="address" property="address"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
        </collection>
    </resultMap>
```

### 4 小结
在这部分中，学习了mybatis关于sql的一些复杂的动态查询，一对一、一对多与多对多等等查询，最基本的方法便是在实体类中把需要关联查询的类作为一个属性，并在配置文件中配置resultMap进行结果映射，最后使用多表查询语句来关联查询，得到需要的结果。  