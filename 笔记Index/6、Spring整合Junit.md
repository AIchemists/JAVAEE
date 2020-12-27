## 【掌握】Spring整合Junit
在上述的代码中，每次进行数据库操作都要先创建一个Service的类。  
```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml"); 
IAccountService as = ac.getBean("accountService",IAccountService.class);
```
junit 给了一个注解，可以用来替换掉它的运行器。只需要我们依靠 spring 框架，因为它提供了一个运行器，可以读取配置文件（或注解）来创建容器。我们只需要告诉它配置文件在哪就行了。  
整体代码如下：  
```java
@RunWith(SpringJUnit4ClassRunner.class) 
@ContextConfiguration(locations= {"classpath:bean.xml"}) 
public class AccountServiceTest 
{
@Autowired 
private IAccountService as ;
}
```
`@RunWith`注解用来替换原有运行器  
`@ContextConfiguration `注解：   
- locations 属性：用于指定配置文件的位置。如果是类路径下，需要用 classpath:表明   
- classes 属性：用于指定注解的类。当不使用 xml 配置时，需要用此属性指定注解类的位置。  

如此一来，就能自动创建容器了。  