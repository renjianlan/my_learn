##### Springboot整合Spring Security(Shiro)

客户端访问的权限控制

1、pom.xml

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-parent</artifactId>
    <version>2.1.11.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

2、创建Handler

```java
@Controller
@RequestMapping("/hello")
public class HelloHandler {

    @GetMapping("index")
    public String index(){
        return "index";
    }
}
```

3、创建Html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>
        Hello World!
    </h1>
</body>
</html>
```

4、创建配置文件

```yaml
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
```

5、创建启动类

6、设置自定义密码

```yaml
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
  security:
    user:
     name: admin
     password: 123123
```

##### 权限管理

定义两个html资源，index.html 、admin.html,同时定义两个角色，admin和user,admin可以访问index.html和admin.html，user只能访问index.html

1、创建MyPasswordEncoder管理类

```java
package example.config;

import org.springframework.security.crypto.password.PasswordEncoder;

public class MyPasswordEncoder implements PasswordEncoder {
    public String encode(CharSequence charSequence) {
        return charSequence.toString();
    }

    public boolean matches(CharSequence charSequence, String s) {
        return s.equals(charSequence.toString());
    }
}

```

2、创建SecurityConfig管理类

```java
package example.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //进行密码编码
        auth.inMemoryAuthentication().passwordEncoder(new MyPasswordEncoder())
                .withUser("user").password(new MyPasswordEncoder().encode("000")).roles("USER")
                .and()
                .withUser("admin").password(new MyPasswordEncoder().encode("123")).roles("USER","ADMIN");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/admin").hasRole("ADMIN")
                .antMatchers("/index").access("hasRole('ADMIN') or hasRole('USER')")
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login")
                .permitAll()
                .and()
                .logout()
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}

```

3、admin.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>后台管理系统</h1>
<form action="/logout" method="post">
    <input type="submit" value="退出"/>
</form>
</body>
</html>
```

4、index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>
        Hello World!
    </h1>
    <form action="/logout" method="post">
        <input type="submit" value="退出"/>
    </form>
</body>
</html>
```

5、login.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"></html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form th:action="@{/login}" method="post">
    用户名：<input type="text" name="username"/></br>
    密码：<input type="text" name="password"/></br>
    <input type="submit" value="登录"/>
</form>
</body>
</html>
```

6、Handler

```java
@Controller
public class HelloHandler {

    @GetMapping("/index")
    public String index(){
        return "index";
    }

    @GetMapping("/admin")
    public String admin(){
        return "admin";
    }

    @GetMapping("/login")
    public String login(){
        return "login";
    }
}

```

7、application.yml

```yaml
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
  security:
    user:
     name: admin
     password: 123123
```

