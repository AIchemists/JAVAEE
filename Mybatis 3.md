### 本篇是本人看完所有Mybatis教程视频后总结的内容
# Mybatis框架学习3——连接池、事务与sql
主要学习内容：  
- Mybatis的连接池与事务控制
- Mybatis的动态sql语句
- Mybatis的多表查询

## 1、	Mybatis的连接池技术
在 Mybatis 的 SqlMapConfig.xml 配置文件中，通过<dataSource type=”pooled”>来实现 Mybatis 中连接池的配置。  
Mybatis的数据源分为三类：  
- UNPOOLED 不使用连接池的数据源 
- POOLED 使用连接池的数据源 
- JNDI 使用 JNDI 实现的数据源
所谓数据源可以理解为使用连接池技术来更好的管理数据库连接。  
PS:在本次课程学习中，采用的都是POOLED数据源。  
image1
