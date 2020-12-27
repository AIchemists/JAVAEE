## Mybatis的动态SQL语句
### 1 <if>标签
满足if条件才执行，比如在 id 如果不为空时可以根据 id 查询， 如果 username 不同空时还要加入用户名作为条件。这种情况在我们的多条件组合查询中经常会碰到。  
```html
<select id="findUserByCondition" resultMap="userMap" parameterType="user">
        select * from user where 1=1
        <if test="userName != null">
          and username = #{userName}
        </if>
        <if test="userSex != null">
            and sex = #{userSex}
        </if>
    </select>
```
在这里使用`where 1=1 `是为了满足多个条件的查询。  
这样，就可以通过<if>标签来查询满足的记录了。  

### 2<where>标签 
为了简化上面 `where 1=1 `的条件拼装，也可以采用<where>标签来简化开发。 
  ```html
<select id="findUserByCondition" resultMap="userMap" parameterType="user">
        select * from user
        <where>
            <if test="userName != null">
                and username = #{userName}
            </if>
            <if test="userSex != null">
                and sex = #{userSex}
            </if>
        </where>
    </select>
  ```
这样就不用添加条件` where 1=1 `了。

### 3<foreach>标签
  ```html
<select id="findUserInIds" resultMap="userMap" parameterType="queryvo">
        <include refid="defaultUser"></include>
        <where>
            <if test="ids != null and ids.size()>0">
                <foreach collection="ids" open="and id in (" close=")" item="uid" separator=",">
                    #{uid}
                </foreach>
            </if>
        </where>
    </select>
  ```
在where中使用<foreach>标签，用于遍历集合元素。  
- collection:代表要遍历的集合元素，注意编写时不要写#{}  
- open:代表语句的开始部分  
- close:代表结束部分  
- item:代表遍历集合的每个元素，生成的变量名  
- sperator:代表分隔符  
  
这条sql语句为：
 ```sql
 select * from user where id in (？);  
```
### 4 简化SQL片段
Mybatis可以将重复的 sql 提取出来，使用时用 include 引用即可，最终达到 sql 重用的目的。  
#### 4.1 定义SQL片段
```html
<sql id="defaultUser">
        select * from user
    </sql>
```
抽取重复的sql语句片段，设置一个id
#### 4.2 引用SQL片段
```html
<select id="findById" parameterType="INT" resultMap="userMap">
        <include refid=”defaultSql”> </include>
where id = #{uid}
    </select>
```
使用<include>标签将sql片段引入进来，得到完整的sql语句。  
  
上述sql语句为
 ```sql
  select * from user where id = #{uid}
```