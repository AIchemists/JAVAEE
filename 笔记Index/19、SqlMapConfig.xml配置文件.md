## SqlMapConfig.xml配置文件
在SqlMapConfig.xml 中有很多可以配置的信息，如属性、别名、环境、映射器等等，视频中主要讲解了三个部分。  
### 1 properties(属性)
在使用 properties 标签配置时，我们可以采用两种方式指定属性配置。  
第一种则是视频中使用的<property>标签，将名称和值一一对应起来。  
```html
 <properties>
       <property name="driver" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/eesy_mybatis"></property>
        <property name="username" value="root"></property>
        <property name="password" value="1234"></property>
    </properties>
```

第二种则是通过读取一个属性配置文件来获取properties信息。在resource中创建一个properties文件比在其中写下配置信息，再用url来定位。  
```html
<properties url="file:///D:/IdeaProjects/day02_eesy_01mybatisCRUD/src/main/resources/jdbcConfig.properties">
    </properties>
```
### 2 typeAliases(类型别名)
在 SqlMapConfig.xml 中配置：  
```html
<typeAliases>
        <typeAlias type="com.itheima.domain.User" alias="user"></typeAlias>
        <package name="com.itheima.domain"></package>
    </typeAliases>
  ```
typeAlias用于配置别名。type属性指定的是实体类全限定类名。  
alias属性指定别名，当指定了别名就不再区分大小写  
package用于指定要配置别名的包，**当指定之后，该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写**

### 3 mappers（映射器）
#### 3.1 <mapper resource=” ”/>
使用配置文件方法来映射Dao接口，配置的内容为配置文件的路径。  
```html
<mapper resource=”com/itheima/dao/IUserDao/xml”/>
```

#### 3.2 <mapper class=” ”/>
使用注解的方法，写入内容指定被注解的dao全限定类名  
```html
<mapper class=”com.itheima.dao.IUserDao”/>
```
### 3 <package name=” ”/>
注册指定包下的所有 mapper 接口  
```html
<package name=”cn.itcast.mybatis.mapper”/>
```
**注意：使用配置文件映射都要求 mapper 接口名称和 mapper 映射文件名称相同，且放在同一个目录中。**