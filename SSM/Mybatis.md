# Mybatis学习

## 1、环境搭建

+ 创建项目，引用Mysql，junit，mybatis包

  ```xml
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.18</version>
  </dependency>
  
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
  </dependency>
  
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
  </dependency>
  ```

  

### 1.2、基础配置

+ 需要在工具类中获取SqlSessionFactory对象，创建SqlSession实例：

  ```java
  String resource = "org/mybatis/example/mybatis-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  
  //获取实例
  sqlSessionFactory.openSession();
  ```

+ XML中构建SqlSessionFactory，配置资源文件mybatis-config.xml：

  配置数据库基本信息，Mapper.xml查询配置文件地址等。

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <environments default="development">
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=GMT"/>
                  <property name="username" value="root"/>
                  <property name="password" value="root"/>
              </dataSource>
          </environment>
      </environments>
      <mappers>
          <mapper resource="com/joker/dao/UserMapper.xml"/>
      </mappers>
  </configuration>
  ```

+ Mapper.xml配置

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="org.mybatis.example.BlogMapper">
    <select id="selectBlog" resultType="Blog">
      select * from Blog where id = #{id}
    </select>
  </mapper>
  ```

  

### 1.3、遇到的问题

+ Maven约定大于配置，资源文件默认只拷resources下的，src/main/java下的资源文件需要配置pom.xml

  ```xml
  <build>
      <resources>
          <resource>
              <directory>src/main/resources</directory>
              <includes>
                  <include>**/*.properties</include>
                  <include>**/*.xml</include>
              </includes>
              <filtering>true</filtering>
          </resource>
          <resource>
              <directory>src/main/java</directory>
              <includes>
                  <include>**/*.properties</include>
                  <include>**/*.xml</include>
              </includes>
              <filtering>true</filtering>
          </resource>
      </resources>
  </build>
  ```

  

  

## 2、CURD

### 2.1、简单的增删改查操作

+ Mapper.xml中配置SQL，可以直接绑定到接口中（代码中传递的对象会自动解析为对象的属性）

  ```xml
      <select id="getUserById" resultType="com.joker.pojo.User">
          select *
          from mybatis.user
          where id = #{id}
      </select>
  
      <insert id="addUser" parameterType="com.joker.pojo.User">
          insert into mybatis.user (id, name, pwd)
          values (#{id}, #{name}, #{pwd})
      </insert>
  
      <update id="updateUser" parameterType="com.joker.pojo.User">
          update mybatis.user
          set name= #{name},
              pwd = #{pwd}
          where id = #{id};
      </update>
  
      <delete id="deleteUser">
          delete
          from mybatis.user
          where id = #{id};
      </delete>
  ```

+ Java代码中可直接传递对象，别忘记提交事务

  ```java
  try (SqlSession sqlSession = MybatisUtils.getSqlSession()) {
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
      //一个参数可以直接传，多个参数用map
      User user = mapper.getUserById(1);
      System.out.println(user);
  }
  
      int row = mapper.addUser(new User(4,"张伟", "zhang123"));
      if(row>=1)
          System.out.println("add success");
  
      sqlSession.commit();
  ```

  

### 2.2、使用万能的Map

+ Mapper配置

  ```xml
  <select id="getUserByMap" parameterType="map" resultType="com.joker.pojo.User">
      <bind name="rightLikeFirstName" value="firstName + '%'"/>
      select *
      from mybatis.user
      where name like #{rightLikeFirstName}
  </select>
  ```

+ 调用如下：

  ```java
  Map<String, Object> map = new HashMap<>();
  map.put("firstName", "张");
  List<User> userList = mapper.getUserByMap(map);
  ```



### 2.3、模糊查询

+ Java代码中使用通配符，直接在参数上拼%

+ Mapper中SQL使用通配符（可以使用bind拼接字符串参数，也可以使用mysql的cancat函数）

  ```xml
  <select id="getUserByMap" parameterType="map" resultType="com.joker.pojo.User">
      <bind name="rightLikeFirstName" value="firstName + '%'"/>
      select *
      from mybatis.user
      where name like #{rightLikeFirstName}
  </select>
  ```

  ```xml
  <select id="getUserByMap" parameterType="map" resultType="com.joker.pojo.User">
      select *
      from mybatis.user
      where name like concat( #{firstName}, '%')
  </select>
  ```

  

### 2.3、遇到的问题

+ 比如mapper.xml配置文件乱码的问题，使用pom.xml中声明properties编码都无法解决，最后用编辑器打开报错xml文件，使用UTF-8重新保存即可。

  ```xml
  <!--不知道为什么不行-->   
  <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      </properties>
  ```

  

## 2、配置解析

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息

```
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

### 2.1、环境配置（environments）

**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

Mybatis默认的事务管理器时JDBC，连接池是POOLED

### 2.2、属性（properties）

**这些属性可以在外部进行配置，并可以进行动态替换。**

+ 比如数据库的属性可以写在一个properties文件中方便后续整理

  ```properties
  driver=com.mysql.cj.jdbc.Driver
  url=jdbc:mysql://localhost:3306/?useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT
  username=root
  password=root
  ```

+ 另外如果mybatis配置中也可以配置属性(properties配置文件优先)

  ```xml
   <properties resource="db.properties">
          <property name="username" value="root"/>
          <property name="password" value="123456"/>
      </properties>
  ```

  

### 2.3、类型别名（typeAliases）

**类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。**

+ 为单个类配置别名（可以自定义别名）

  ```xml
      <typeAliases>
          <typeAlias type="com.joker.pojo.User" alias="User"/>
      </typeAliases>
  ```

+ 扫描整个包(使用时直接使用类名即可，类名的小写就可以，不可以自定义，但是可以用@Aliase注解定义别名)

  ```xml
      <typeAliases>
          <package name="com.joker.pojo"/>
      </typeAliases>
  ```

  + Java中注解自定义别名(配置过扫描包之后才有效)

    ```java
    @Alias("user")
    public class User
    ```



### 2.4、映射器（mappers）

**MapperRegistry：注册绑Mapper映射文件**

+ 使用相对于类路径的资源引用【推荐使用】

  ```xml
      <mappers>
          <mapper resource="com/joker/dao/UserMapper.xml"/>
      </mappers>
  ```

+ 使用映射器接口实现类的完全限定类名（对象是映射器）

  ```xml
      <mappers>
          <mapper class="com.joker.dao.UserMapper"/>
      </mappers>
  ```

+ 将包内的映射器接口实现全部注册为映射器（都是处理映射器）

  ```xml
      <mappers>
          <package name="com.joker.dao"/>
      </mappers>
  ```

  **注意点：**

  后两种映射器，接口和他的Mapper配置文件必须同名！且位于同一个包下！



### 2.5、作用域（Scope）和生命周期

todo



### 2.6、结果映射

**`resultMap`元素是 MyBatis 中最重要最强大的元素。**

+ 最简单的映射：

  ```xml
      <resultMap id="UserMap" type="user">
  <!--        <result column="id" property="id"/>-->
  <!--        <result column="name" property="name"/>-->
          <result column="pwd" property="password"/>
      </resultMap>
  ```

  + 同时sql映射改为(注意是resultMap改为上面的映射id)：

    ```xml
        <select id="getUserById" resultMap="UserMap">
            select * from mybatis.user where id = #{id}
        </select>
    ```

+ 列明和属性名一致的不需要映射

### 2.7、高级结果映射





## 3、日志

mybatis-04

### 3.1、日志工厂

**logImpl**：指定 MyBatis 所用日志的具体实现，未指定时将自动查找。（setting设置）

​		SLF4J | LOG4J | LOG4J2 | JDK_LOGGING | COMMONS_LOGGING | STDOUT_LOGGING | NO_LOGGING

+ **STDOUT_LOGGING：**标准日志工厂实现 (可以直接使用该日志工厂，只需配置setting的logImpl)

  ```xml
      <settings>
          <setting name="logImpl" value="STDOUT_LOGGING"/>
      </settings>
  ```

+ **LOG4J：** P12，有时间再看



## 4、分页

+ 使用limit进行分页

  ```xml
      <select id="getUserListLimit" parameterType="map" resultMap="UserMap">
          select * from mybatis.user limit #{startIndex}, #{pageSize}
      </select>
  ```

  