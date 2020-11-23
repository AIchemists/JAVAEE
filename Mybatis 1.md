### 本篇是本人看完所有Mybatis教程视频后的总结笔记
# Mybatis框架学习1——Mybatis入门
## 主要学习内容：
- Mybatis框架的配置文件实现
- Mybatis框架的注解实现
- 自定义类来实现Mybatis框架

## 1、	Mybatis框架概述
Mybatis是一个基于java的优秀的持久层框架，所谓持久层框架，即负责与数据库的数据交互的框架。
Mybatis 通过 xml 配置文件或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 sql 的动态参数进行映射生成最终执行的sql语句，最后由 Mybatis框架执行 sql 并将结果映射为 java 对象并返回。
同时，Mybatis采用 ORM 思想解决了程序实体和数据库映射的问题，对 jdbc 进行了封装，使我们不用与 jdbc api 打交道，就可以完成对数据库的持久化操作，即实现实体类与数据库表的一一对应与转化。
Mybatis的优点：  
1、	Mybatis使开发者更专注于sql语句本身这也是视频中老师强调的最多的一点。sql写在xml里，便于统一管理和优化。开发者不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。  
2、	Mybatis提供映射标签，支持对象与数据库的ORM字段关系映射,而不用开发者手动转化数据。  
3、	Mybatis提供了各种各样的缓存、延迟加载机制，提高对数据库查询的效率，以提高应用的性能。  
## 2、Mybatis的xml配置实现
### 2.1传统jdbc
传统jdbc程序执行sql语句的步骤是：
- 加载数据库驱动
`Class.forName("com.mysql.jdbc.Driver");`
- 通过驱动管理类获取数据库连接
 `connection = DriverManager .getConnection("jdbc:mysql://localhost:3306/Mybatis?characterEncoding=utf-8","root", "root");`
- 定义sql语句
 `String sql = "select * from user where username = ?";`
- 获取预处理
 `statement preparedStatement = connection.prepareStatement(sql);`
- 设置sql参数
 `preparedStatement.setString(1, "王五");`
- 向数据库发出sql执行语句
 `resultSet = preparedStatement.executeQuery(); `
- 遍历查询结果集
 `while(resultSet.next()){ System.out.println(resultSet.getString("id")+" "+resultSet.getString("username")); } `
- 释放资源
 `resultSet.close(); preparedStatement.close(); connection.close();` 
 
可以看出，传统的jdbc方法有很多问题：  
1、数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。   
2、Sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 java 代码。   
3、使用 preparedStatement 向占有位符号传参数存在硬编码，因为 sql 语句的 where 条件不一定，可能 多也可能少，修改 sql 还要修改代码，系统不易维护。  
4、对结果集解析存在硬编码（查询列名），sql 变化导致解析代码变化，系统不易维护，如果能将数据库记 录封装成 pojo 对象解析比较方便。

### 2.2 Mybatis框架配置实现
#### 2.2.1 持久层接口与映射文件
在下载了Mybatis的jar包并添加到pom.xml后，编写持久层的接口



