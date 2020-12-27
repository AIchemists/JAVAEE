## Mybatis 的事务控制
### 1 自动提交事务和手动提交事务
在之前的手动提交事务时，需要在sql的CUD操作执行后调用  
`sqlsession.commit();`
来进行事务的提交，否则对数据库进行的操作无法提交到数据库。  
Mybatis框架的事务控制方式，本身是用 JDBC 的`setAutoCommit()`方法来设置事务提交方式的。  
image3  
在自动提交模式下，所有的Sql语句被执行时作为单个事务提交。  

### 2 设置自动提交事务
事实上，需要执行sqlsession.commit();来手动提交事务的原因是我们每次从连接池中取出连接，都会将调用` connection.setAutoCommit(false)`方法，而设置自动提交事务的方法即在创建Sqlsession时，传入一个true作为参数：  
`session = factory.openSession(true);`  
这个参数传入后，会将autoCommit的值设为true并传给连接。  
image4  
此时，不需要再手动提交事务sqlsession.commit();事务也能自动的提交了。  