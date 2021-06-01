Spring Boot整合MyBatis

1、创建Maven工程，pom.xml

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-parent</artifactId>
    <version>2.1.11.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.1</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.13</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

```

2、创建数据表

3、创建对应的实体类

```java
@Data
public class User {
    private int id;
    private String name;
    private int age;
    private Double score;
    private Date birthday;
}
```

4、创建UserRepository接口

```java
@Repository
public interface UserRepository {
    public List<User> findAll();
    public User findById(int id);
    public void save(User user);
    public void update(User user);
    public void deleteById(int id);
}
```

5、在resources/mapping路径下创建UserRepository接口对应的Mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="example.repository.UserRepository">
    <select id="findAll" resultType="User">
        select * from user
    </select>

    <select id="findById" parameterType="java.lang.Integer" resultType="User">
        select * from user where id=#{id}
    </select>

    <insert id="save" parameterType="User">
        insert into user(name,age,score,birthday) values (#{name},#{age},#{score},#{birthday})
    </insert>

    <update id="update" parameterType="User">
        update user set name=#{name},age=#{age} where id=#{id}
    </update>

    <delete id="deleteById" parameterType="java.lang.Integer">
        delete from user where id=#{id}
    </delete>
</mapper>
```

6、创建UserHandler

```java
@RestController
@RequestMapping("/user")
public class UserHandler {
    @Autowired
    private UserRepository userRepository;

    @GetMapping("/findAll")
    public List<User> findAll(){
        return userRepository.findAll();
    }

    @GetMapping("/findById/{id}")
    public User findById(@PathVariable("id") int id){
        return userRepository.findById(id);
    }

    @PostMapping("/save")
    public void save(@RequestBody User user){
        userRepository.save(user);
    }

    @PutMapping("/update")
    public void update(@RequestBody User user){
        userRepository.update(user);
    }

    @DeleteMapping("/deleteById/{id}")
    public void deleteById(@PathVariable("id") int id){
        userRepository.deleteById(id);
    }
}
```

7、application.yml

```xml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
    username: root
    password: "0123456789"
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
  mapper-locations: classpath:/mapping/*.xml
  type-aliases-package: example.entity //mapper.xml中实体类前面公共包名
```



