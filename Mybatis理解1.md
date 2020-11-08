# Mybatis
## 代理DAO
### 自定义类
最终工具类Executor, Executor需要mapper和connection，其中mapper记录查询语句和返回数据类型，connection创建数据库连接。在代理对象MapperProxy中增强调用Executor，增强方法是获取mapper，mapper的获取方法是由工具类XMLConfigBuilder来读取配置文件，并存入Configuration对象中。  总的来说就两件事：查询所有和创建代理对象调用查询所有操作。
