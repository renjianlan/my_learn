#### Spring MVC 

Spring MVC是目前主流的实现MVC设计模式的企业级开发框架，Spring框架的一个子模块，无需整合，开发起来更加便捷

##### 什么是MVC设计模式

将应用程序分为Controller、Model、View三层，Controller接收客户请求，调用Model生成业务数据，传递给View

SpringMVC就是对这套流程的封装，屏蔽了很多底层代码，开放出接口，让开发者可以更加轻松、便捷地完成基于MVC模式的web开发

##### SpringMVC的核心组件

- DispatcherServlet：前置控制器，是整个流程控制的核心，控制其他组件的执行，进行统一调度，降低组件之间的耦合性相当于总指挥
- Handler：处理器，完成具体的业务逻辑，相当于Servlet或Action
- HandlerMapping：DispatcherServlet接收到请求之后，通过HandlerMapping将不同的请求映射到不同的Handler.
- HandlerInterceptor：处理器拦截器，是一个接口，如果需要完成一些拦截处理，那么就可以实现该接口
- HandlerExecutionChain：处理器执行链，包括两部分内容：Handler和HandlerInterceptor（系统会有默认的HandlerInterceptor，如果需要额外设置拦截，可以添加拦截器）
- HandlerAdapter：处理器适配器，Handler执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到JavaBean等，这些操作都是由HandlerAdapter来完成，DispatcherServlet通过HandlerAdapter执行不同的Handler
- ModelAndView：装载了模型数据和视图信息，作为Handler的处理结果，返回给DispatcherServlet
- ViewResolver：视图解析器，DispatcherServlet通过它将逻辑视图解析为物理视图，将渲染的结果响应给客户端

##### SpringMVC的工作流程(真正需要开发者进行处理的只有View和Handler)

- 客户端请求被DispatcherServlet接收
- 根据HandlerMapping映射到Handler
- 生成HandlerInterceptor和Handler
- HandlerInterceptor和Handler以HandlerExecutionChain的形式一并返回给DispatcherServlet
- DispatcherServlet通过HandlerAdapter来调用Handler的方法，完成业务逻辑处理
- Handler返回一个ModelAndView给DispatcherServlet
- DispatcherServlet将获取的ModelAndView传给ViewResolver视图解析器，将逻辑视图（只是一个视图名）解析为物理视图View
- ViewResolver返回一个DispatcherServlet
- DispatcherServlet根据View进行视图渲染，将模型数据Model填充到视图View中，最后DispatcherServlet将渲染后的结果响应到客户端

##### 如何使用？

- 创建Maven工程，pom.xml

  ```xml
  <dependencies>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.0.11.RELEASE</version>
      </dependency>
    </dependencies>
  ```

- 在web.xml中配置DispatcherServlet

  ```xml
  <!DOCTYPE web-app PUBLIC
   "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
   "http://java.sun.com/dtd/web-app_2_3.dtd" >
  
  <web-app>
    <display-name>Archetype Created Web Application</display-name>
    <servlet>
      <servlet-name>dispatcherServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
      </init-param>
    </servlet>
    <servlet-mapping>
      <servlet-name>dispatcherServlet</servlet-name>
      <url-pattern>/</url-pattern>
    </servlet-mapping>
  
  </web-app>
  ```

- springmvc.xml

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:p="http://www.springframework.org/schema/p"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
      					http://www.springframework.org/schema/beans/spring-beans.xsd
      					http://www.springframework.org/schema/context
      					http://www.springframework.org/schema/context/spring-context.xsd">
      <!--自动扫描-->
      <context:component-scan base-package="com.example"></context:component-scan>
  
      <!--配置视图解析器-->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <property name="suffix" value=".jsp"></property>
          <property name="prefix" value="/"></property>
      </bean>
  </beans>
  ```

  ##### SpringMVC注解

- @RequestMapping

  SpringMVC通过@RequestMapping注解将URL请求与业务方法进行映射，在Handler的类定义处以及方法定义处都可以添加该注解，在类的定义处添加就相当于客户端多了一层访问路径

- @Controller

  在类的定义处添加，将该类交给ioc容器管理（结合springmvc.xml的自动扫描配置使用），使其成为控制器，可以接收客户端请求

- @RequestMapping相关参数

  - value：指定url请求的实际地址，是@RequestMapping的默认值

    @RequestMapping（value="/index") 和@RequestMapping("/index")功能一样

  - method：指定请求的method类型，GET、POST、PUT、DELETE，不加请求类型代表GET和POST都可以

  - params：指定请求中必须包含某些参数才能访问，例params={"name","id=10"}

  - @RequestParam("name")String str完成HTTP请求参数与业务方法形参的映射，自动完成数据类型的转换，这些工作都是由HandlerAdapter完成

   SpringMVC也支持RESTful风格的url

- 传统类型：http://localhost:8080/hello/index?name=zhangsan&id=10

- REST：http://localhost:8080/hello/rest/zhangsan/10，必须加注解映射，通过@PathVariable注解完成请求参数与形参的映射

```java
@RequestMapping("/rest/{name}/{id}")
public String rest(@PathVariable("name")String name, @PathVariable("id")int id){
    System.out.println(name);
    System.out.println(id);
    return "index";
}
```

- 映射Cookie

  Spring MVC通过映射可以直接在业务方法中获取Cookie的值

  ```java
  @RequestMapping("/cookie")
  public String cookie(@CookieValue(value="JSESSIONID") String sessionId){
      System.out.println(sessionId);
      return "index";
  }
  ```

- 使用javaBean绑定参数：SpringMVC会根据请求的参数名和javaBean的属性名进行自动匹配，自动为对象填充属性值，同时支持级联的属性。如果出现中文乱码的问题，需要在web.xml中添加Spring  MVC自带的过滤器即可

  ```xml
  <filter>
      <filter-name>encodingFilter</filter-name>
      <filter-class>
          org.springframework.web.filter.CharacterEncodingFilter
      </filter-class>
      <init-param>
          <param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
      </init-param>
  </filter>
  
  <filter-mapping>
      <filter-name>encodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

- JSP页面的转发和重定向

Spring MVC默认是以转发的形式相应JSP

1、转发

```java
@RequestMapping("/forward")
public String forward(){
    return "forward:/index.jsp";
    //       return "index";
}
```

2、重定向

```java
@RequestMapping("/redirect")
public String redirect(){
    return "redirect:/index.jsp";
}
```

##### Spring MVC数据绑定

数据绑定：在后端的业务方法中，直接获取客户端HTTP请求中的参数，将请求参数映射到业务方法的形参中，Spring MVC中数据绑定的工作是由HandlerAdapter来完成的

- 基本数据类型

  ```java
  @RequestMapping("/baseType")
  @ResponseBody
  public String baseType(int id){
      return id+"";
  }
  ```

  @ResponseBody表示Spring MVC会直接将业务方法的返回值响应给客户端，如果不加该注解，Spring MVC会将业务方法的返回值传递给DispatcherServlet，再由DispatcherServlet调用ViewResolver对返回值进行解析，映射到一个JSP资源

- 包装类

  ```java
  @RequestMapping("/packageType")
  @ResponseBody
  public String packageType(Integer id){
     return id+"";
  }
  ```

  包装类可以接收null，当HTTP请求中没有参数时，使用包装类定义形参的数据类型，程序不会抛出异常
  
  @RequestParam参数
  
  value="num"：将HTTP请求中名为num的参数赋给形参id
  
  required：设置num是否为必填项，true表示必填，false表示非必填，可省略
  
  defaultValue=“0”：如果HTTP请求中没有num参数，默认值为0
  
  ```java
  
  @RequestMapping("/packageType")
  @ResponseBody
  public String packageType(
      @RequestParam(value="num",required=false,defaultValue = "0") Integer id){
      return id+"";
  }
  ```

- 数组

  ```java
  @RequestMapping("/array")
  @ResponseBody
  public String array(String[] name){
      String str= Arrays.toString(name);
      return str;
  }
  ```

  @RestController表示该控制器直接将业务方法的返回值响应给客户端，不进行视图解析，@Controller表示该控制器的每一个业务方法的返回值都会交给视图解析器进行解析，如果只需要将数据响应给客户端不需要进行视图解析则需要在对应的业务方法定义处添加@ResponseBody注解

  ```java
  @Controller
  @RequestMapping("/data")
  public class DataBindHandler {
      @RequestMapping("/array")
      @ResponseBody
      public String array(String[] name){
          String str= Arrays.toString(name);
          return str;
      }
  }
  ```

  等同于

  ```java
  @RestController
  @RequestMapping("/data")
  public class DataBindHandler {
      @RequestMapping("/array")
      public String array(String[] name){
          String str= Arrays.toString(name);
          return str;
      }
  }
  ```

- List

  SpringMVC不支持List类型的直接转换，需要对List集合进行包装

  ```java
  @Data
  public class UserList {
      private List<User> userList;
  }
  ```

  ```jsp
  <form action="/data/list" method="post">
      用户1编号:<input type="text" name="userList[0].id"/></br>
      用户1名称:<input type="text" name="userList[0].name"/></br>
      用户2编号:<input type="text" name="userList[1].id"/></br>
      用户2名称:<input type="text" name="userList[1].name"/></br>
      <input type="submit" value="提交"/>
  </form>
  ```

  ```java
  @RequestMapping("/list")
  @ResponseBody
  public String list(UserList userList){
      String str= "";
      for(User user:userList.getUserList()){
      str+=user;
  }
  	return str;
  }
  ```

  处理@ResponseBody中文乱码，在springmvc.xml中配置消息转换器

```xml
<mvc:annotation-driven>
        <!--消息转换器,后台向前台传数据出现乱码的转换-->
        <mvc:message-converters register-defaults="true">
            <bean          class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes"  value="text/html;charset=UTF-8">

                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

- Map

自定义一个封装类

```java
package com.example.controller.entity;

import java.util.Map;

public class UserMap {
    private Map<String,User> users;
}
```

业务方法

```java
 @RequestMapping("/map")
 @ResponseBody
 public String map(UserMap userMap){
     StringBuffer str= new StringBuffer();
     for(String key:userMap.getUsers().keySet()){
         User user=userMap.getUsers().get(key);
         str.append(user);
     }
     	return str.toString();
 }
```

- JSON

客户端发生JSON格式的数据，直接通过Spring MVC绑定到业务方法的形参中

处理Spring MVC无法加载静态资源，在web.xml中添加配置即可

```xml
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.js</url-pattern>
</servlet-mapping>
```

jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: admin
  Date: 2021/2/12
  Time: 17:01
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
    <script type="text/javascript">
        $(function () {
          var user={
              id:1,
              name:"zhangsan"
          };
          $.ajax({
              url:"/data/json",
              data:JSON.stringify(user),
              type:"POST",
              contentType:"application/json;charset=UTF-8",
              dataType:"JSON",
              success:function (data) {
                  alert(data.id+","+data.name);
              }
          })
        });
    </script>
</head>
<body>

</body>
</html>

```

业务方法

```java
@RequestMapping("/json")
@ResponseBody
public User json(@RequestBody User user){
    System.out.println(user);
    user.setId(6);
    user.setName("李四");
    System.out.println(user);
    return user;
}
```

spring mvc中的JSON和JavaBean的转换需要借助于fastjson，pom.xml引入相关依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.32</version>
</dependency>
```

springmvc.xml中添加fastjson的配置

```xml
   <mvc:annotation-driven>
        <!--消息转换器,后台向前台传数据出现乱码的转换-->
        <mvc:message-converters register-defaults="true">
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes" value="text/html;charset=UTF-8">

                </property>
            </bean>
            <!--配置fastjson-->
            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4"></bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

##### Spring MVC模型数据解析

JSP的四大作用域对应的内置对象：pageContext、request、session、application

- pageContext对象的范围只适用于当前页面范围，即超过这个页面就不能够使用了。所以使用pageContext对象向其它页面传递参数是不可能的。
- request对象的范围是指在一JSP网页发出请求到另一个JSP网页之间，随后这个属性就失效
- session的作用范围为一段用户持续和服务器所连接的时间，但与服务器断线后，这个属性就无效。比如断网或者关闭浏览器。（可以设置超时）
- application的范围在服务器一开始执行服务，到服务器关闭为止。它的范围最大，生存周期最长。

模型数据的绑定是由ViewResolver来完成的，实际开发中，我们需要先添加模型数据，再交给ViewResolver来绑定

Spring MVC提供了以下几种方式添加模型数据：

- Map
- Model
- ModelAndView
- @SeesionAttribute
- @ModelAttribute

Map

```java
@RequestMapping("/map")
public String map(Map<String,User> map){
    User user=new User();
    user.setId(1);
    user.setName("zhangsan");
    map.put("user",user);
    return "view";
}
```

jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: admin
  Date: 2021/2/13
  Time: 10:32
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${requestScope.user}
</body>
</html>

```

model

```java
    @RequestMapping("/model")
    public String model(Model model){
        User user=new User();
        user.setId(1);
        user.setName("zhangsan");
        model.addAttribute("user",user);
        return "view";
    }
```

modelAndView

```java
 @RequestMapping("/modelAndView")
    public ModelAndView modelAndView(){
        User user=new User();
        user.setId(1);
        user.setName("zhangsan");
        ModelAndView modelAndView=new ModelAndView();
        modelAndView.addObject("user",user);
        modelAndView.setViewName("view");
        return modelAndView;
    }

    @RequestMapping("/modelAndView2")
    public ModelAndView modelAndView2(){
        User user=new User();
        user.setId(2);
        user.setName("zhangsan");
        ModelAndView modelAndView=new ModelAndView();
        modelAndView.addObject("user",user);
        View view=new InternalResourceView("/view.jsp");
        modelAndView.setView(view);
        return modelAndView;
    }

    @RequestMapping("/modelAndView3")
    public ModelAndView modelAndView3(){
        User user=new User();
        user.setId(3);
        user.setName("zhangsan");
        ModelAndView modelAndView=new ModelAndView("view");
        modelAndView.addObject("user",user);
        return modelAndView;
    }

    @RequestMapping("/modelAndView4")
    public ModelAndView modelAndView4(){
        User user=new User();
        user.setId(4);
        user.setName("zhangsan");
        View view=new InternalResourceView("/view.jsp");
        ModelAndView modelAndView=new ModelAndView(view);
        modelAndView.addObject("user",user);
        return modelAndView;
    }

    @RequestMapping("/modelAndView5")
    public ModelAndView modelAndView5(){
        User user=new User();
        user.setId(5);
        user.setName("zhangsan");
        Map<String,User> map=new HashMap<>();
        map.put("user",user);
        ModelAndView modelAndView=new ModelAndView("view",map);
        return modelAndView;
    }

    @RequestMapping("/modelAndView6")
    public ModelAndView modelAndView6(){
        User user=new User();
        user.setId(6);
        user.setName("zhangsan");
        Map<String,User> map=new HashMap<>();
        map.put("user",user);
        View  view=new InternalResourceView("/view.jsp");
        ModelAndView modelAndView=new ModelAndView(view,map);
        return modelAndView;
    }

    @RequestMapping("/modelAndView7")
    public ModelAndView modelAndView7(){
        User user=new User();
        user.setId(7);
        user.setName("zhangsan");
        ModelAndView modelAndView=new ModelAndView("view","user",user);
        return modelAndView;
    }
    @RequestMapping("/modelAndView8")
    public ModelAndView modelAndView8(){
        User user=new User();
        user.setId(8);
        user.setName("zhangsan");
        View view=new InternalResourceView("/view.jsp");
        ModelAndView modelAndView=new ModelAndView(view,"user",user);
        return modelAndView;
    }
```

HttpServerletRequest

```java
 @RequestMapping("/request")
    public String request(HttpServletRequest request){
        User user=new User();
        user.setId(1);
        user.setName("zhangsan");
        request.setAttribute("user",user);
        return "view";
    }
```

@ModelAttribute

- 定义一个方法，该方法专门用来返回要填充到模型数据中的对象

  ```java
  @ModelAttribute
      public User getUser(){
          User user=new User();
          user.setId(1);
          user.setName("zhangsan");
          return user;
      }
   或者
   @ModelAttribute
      public void getUser(Map<String,User> map){
          User user=new User();
          user.setId(1);
          user.setName("zhangsan");
          map.put("user",user);
      }
  ```

- 业务方法中无需处理模型数据，只需返回视图即可

  ```java
  @RequestMapping("/modelAttribute")
      public String modelAttribute(){
          return "view";
      }
  ```

  ##### 将模型数据绑定到session对象

1. 直接使用原生的ServletAPI

   ```java
       @RequestMapping("/session")
       public String session(HttpServletRequest request){
           HttpSession session=request.getSession();
           User user=new User();
           user.setId(2);
           user.setName("zhangsan");
           session.setAttribute("user",user);
           return "view";
       }
       @RequestMapping("/session2")
       public String session2(HttpSession session){
           User user=new User();
           user.setId(2);
           user.setName("zhangsan");
           session.setAttribute("user",user);
           return "view";
       }
   ```

2. @SessionAttribute

   ```
   @SessionAttributes(value="user")
   @SessionAttributes(value={"user","address"})
   ```

   对于ViewHandler中的所有业务方法，只要向request中添加了key="user"、key="address"的对象时，SpringMVC会自动将该数据添加到session中，保存key值不变

   ```
   @SessionAttributes(types={"User.class","Address.class"})
   ```

   对于ViewHandler中的所有业务方法，只要向request中添加了数据类型是User、Address的对象时，SpringMVC会自动将该数据添加到session中，保存key值不变

#####        将模型数据绑定到Application对象（只能通过request间接绑定，因为ServletContext没有无参构造函数）

```java
    @RequestMapping("/application")
    public String application(HttpServletRequest request){
        ServletContext application=request.getServletContext();
        User user=new User();
        user.setId(3);
        user.setName("zhangsan");
        application.setAttribute("user",user);
        return "view";
    }
```

##### SpringMVC自定义数据转换器

 数据转换器是指将客户端HTTP请求中的参数转换为业务方法中定义的形参，自定义表示开发者可以自主设计转换的方式，HandlerAdapter已经提供了通用的转换，String转int，String转double，表单数据的封装等，但是在特殊的业务场景下，HandlerAdapter无法进行转换，就需要开发者自定义转换器

客户端输入String类型的数据“2021-02-13”，自定义转换器将该数据装换为Date类型的对象

- 创建DateConverter转换器

  ```java
  package com.example.controller;
  
  import org.springframework.core.convert.converter.Converter;
  
  import java.text.ParseException;
  import java.text.SimpleDateFormat;
  import java.util.Date;
  
  public class DateConverter implements Converter<String,Date> {
  
      private String pattern;
      //bean中需要传参数，日期格式
      public DateConverter(String pattern){
          this.pattern=pattern;
      }
  
      @Override
      public Date convert(String s) {
         SimpleDateFormat simpleDateFormat=new SimpleDateFormat(this.pattern);
         Date date=null;
         try {
             //将字符串按照参数格式转换为日期类型
              date=simpleDateFormat.parse(s);
         } catch (ParseException e) {
              e.printStackTrace();
         }
         return date;
      }
}
  ```

  springmvc.xml中配置转换器
  
  ```xml
  <!--配置自定义转换器-->
      <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
          <property name="converters">
              <list>
                  <bean class="com.example.converter.DateConverter">
                      <constructor-arg type="java.lang.String" value="yyy-MM-dd"></constructor-arg>
                  </bean>
              </list>
          </property>
      </bean>
  
    <mvc:annotation-driven conversion-service="conversionService">
  ```

  Handler
  
  ```java
  @Controller
  @RequestMapping("/converter")
  public class ConvertHandler {
  
      @RequestMapping("/date")
      @ResponseBody
      public String date(Date date){
          return date.toString();
      }
}
  ```
  
  ##### Spring MVC REST
  
  REST:Representational State Transfer 资源表现层状态转换，是目前比较主流的一种互联网软件架构，它结构清晰，标准规范，易于理解，便于扩展。

- 资源（Resource）

  网络上的一个实体，或者说网络中存在的一个具体信息，一段文本、一张图片、一首歌曲、一段视频等等，总之就是一个具体的存在，可以用一个URI（统一资源定位符）指向它，每个资源都有对应的一个特定的URI，要获取该资源的时候只需要访问对应的URI即可

- 表现层（Representation）

  资源具体呈现出来的形式，比如文本可以用txt格式表示，也可用HTML、XML、JSON等格式来表示

- 状态转换（State Transfer)

  客户端如果要操作服务端中的某个资源，就需要通过某种方式让服务端发生状态转化，而这种转换是建立在表现层之上，所有叫做“表现层状态转换”

###### 特点

- URI更加简洁
- 有利于不同系统之间的资源共享，只需要遵守一定的规范，不需要进行其他配置可实现资源共享

###### 如何使用()

REST具体操作就是HTTP协议中四个表示操作方式的动词分别对应CRUD基本操作

GET用来表示获取资源

POST用来表示新建资源

PUT用来表示修改资源

DEETE用来表示删除资源

Handler

```java
package com.example.controller;

import com.example.entity.Student;
import com.example.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletResponse;
import java.util.Collection;

@RestController
@RequestMapping("/rest")
public class RESTHandler {

    @Autowired
    private StudentRepository studentRepository;

    //@RequestMapping(value="/findAll",method= RequestMethod.GET)
    @GetMapping("/findAll")
    public Collection<Student> findAll(HttpServletResponse response){
        response.setContentType("");
        return studentRepository.findAll();
    }
    @GetMapping("/findById/{id}")
    public Student findById(@PathVariable("id")int id, HttpServletResponse response){
        response.setContentType("");
        return studentRepository.findById(id);
    }
    @PostMapping("/save")
    public void save(@RequestBody Student student){
        studentRepository.saveOrUpdate(student);
    }
    @PutMapping("/update")
    public void update(@RequestBody Student student){
        studentRepository.saveOrUpdate(student);
    }
    @DeleteMapping("/delete/{id}")
    public void delete(@PathVariable("id") int id){
        studentRepository.deleteById(id);
    }
}

```

StudentRepository

```java
package com.example.repository;

import com.example.entity.Student;

import java.util.Collection;

public interface StudentRepository {
    public Collection<Student> findAll();
    public Student findById(int id);
    public void saveOrUpdate(Student student);
    public void deleteById(int id);
}
```

StudentRepositoryImpl

```java
package com.example.repository.impl;

import com.example.entity.Student;
import com.example.repository.StudentRepository;
import org.springframework.stereotype.Repository;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

@Repository
public class StudentRepositoryImpl implements StudentRepository {
    private static Map<Integer,Student> studentMap;

    static{
        studentMap=new HashMap<>();
        studentMap.put(1,new Student(1,"zhangsan",22));
        studentMap.put(2,new Student(2,"lisi",23));
        studentMap.put(3,new Student(3,"wangwu",24));
    }

    @Override
    public Collection<Student> findAll() {
        return studentMap.values();
    }

    @Override
    public Student findById(int id) {
        return studentMap.get(id);
    }

    @Override
    public void saveOrUpdate(Student student) {
        studentMap.put(student.getId(),student);
    }

    @Override
    public void deleteById(int id) {
         studentMap.remove(id);
    }
}
```

##### Spring MVC文件上传下载

- 单文件上传

  底层是使用Apache fileupload组件完成上传，SpringMVC对这种方式进行了封装

  1. pom.xml

     ```xml
         <dependency>
           <groupId>commons-io</groupId>
           <artifactId>commons-io</artifactId>
           <version>2.5</version>
         </dependency>
     
         <dependency>
           <groupId>commons-fileupload</groupId>
           <artifactId>commons-fileupload</artifactId>
           <version>1.3.3</version>
         </dependency>
     ```

  2. JSP

     ```jsp
     <%@ page contentType="text/html;charset=UTF-8" language="java" %>
     <html>
     <head>
         <title>Title</title>
     </head>
     <body>
     <form action="/file/upload" method="post" enctype="multipart/form-data">
         <input type="file" name="img"/></br>
         <input type="submit" value="上传"/>
     </form>
     <img src="${path}">
     </body>
     </html>
     
     input的type为file
     form的method设置为post（get请求只能将文件名传给服务器）
     form的enctype设置为multipart-form-data（如果不设置只能将文件名传给服务器）
     ```

  3. Handler

     ```java
     package com.example.controller;
     
     import org.springframework.stereotype.Controller;
     import org.springframework.web.bind.annotation.PostMapping;
     import org.springframework.web.bind.annotation.RequestMapping;
     import org.springframework.web.multipart.MultipartFile;
     
     import javax.servlet.http.HttpServletRequest;
     import java.io.File;
     import java.io.IOException;
     
     @Controller
     @RequestMapping("/file")
     public class FileHandler {
         @PostMapping("/upload")
         public String upload(MultipartFile img, HttpServletRequest request){
             System.out.println(img);
             if(img.getSize()>0){
                 //获取保存上传文件的file路径(服务器上保存图片的文件夹）
                 String path=request.getServletContext().getRealPath("file");
                 //获取上传文件的文件名
                 String name=img.getOriginalFilename();
                 File file=new File(path,name);
                 try {
                     //将img内容转到file中
                     img.transferTo(file);
                     //保存上传之后的文件路径
                     request.setAttribute("path","/file/"+name);
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
             return "upload";
         }
     }
     ```

  4. springmvc.xml

     ```xml
     <!--文件上传配置-->
     <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
     ```

  5. web.xml添加如下配置，否则客户端无法访问jpg

     ```xml
       <servlet-mapping>
         <servlet-name>default</servlet-name>
         <url-pattern>*.jpg</url-pattern>
       </servlet-mapping>
     ```

- 多文件上传

   pom.xml

  ```xml
     <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
      </dependency>
  
      <dependency>
        <groupId>taglibs</groupId>
        <artifactId>standard</artifactId>
        <version>1.1.2</version>
      </dependency>
  ```

  JSP

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <%@ page isELIgnored="false" %>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  <form action="/file/uploads" method="post" enctype="multipart/form-data">
      file1:<input type="file" name="imgs"/></br>
      file2:<input type="file" name="imgs"/></br>
      file3:<input type="file" name="imgs"/></br>
      <input type="submit" value="上传"/>
  </form>
  <c:forEach items="${filesPath}" var="filePath">
      <img src="${filePath}" width="300px">
  </c:forEach>
  </body>
  </html>
  ```

  Handler

  ```java
  @PostMapping("/uploads")
      public String uploads(MultipartFile[] imgs, HttpServletRequest request){
          List<String> filesPath=new ArrayList<>();
          for(MultipartFile img:imgs){
              if(img.getSize()>0) {
                  //获取保存上传文件的file路径(服务器上保存图片的文件夹）
                  String path = request.getServletContext().getRealPath("file");
                  //获取上传文件的文件名
                  String name = img.getOriginalFilename();
                  File file = new File(path, name);
                  try {
                      //将img内容转到file中
                      img.transferTo(file);
                      //保存上传之后的文件路径
                      request.setAttribute("path", "/file/" + name);
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
                  filesPath.add("/file/"+name);
              }
          }
          request.setAttribute("filesPath",filesPath);
          return "uploads";
      }
  ```

##### SpringMVC下载

一个File对象代表硬盘中实际存在的一个文件或者目录。如果没有就新建一个，如果有就拿到这个file对象

JSP

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
   <a href="/file/download/1">1.jpg</a>
   <a href="/file/download/2">2.jpg</a>
   <a href="/file/download/3">3.jpg</a>
</body>
</html>

```

Handler

```java
@GetMapping("/download/{name}")
    public void download(@PathVariable("name") String name, HttpServletRequest request, HttpServletResponse response){
        if(name!=null){
            name+=".jpg";
            String path=request.getServletContext().getRealPath("file");
            File file=new File(path,name);
            OutputStream outputStream=null;
            if(file.exists()){
                response.setContentType("application/forc-download");
                response.setHeader("Content-Disposition", "attachment; filename=" + name);
                try {
                    outputStream=response.getOutputStream();
                    outputStream.write(FileUtils.readFileToByteArray(file));
                    outputStream.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }finally {
                    if(outputStream!=null){
                        try {
                            outputStream.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }

            }
        }
    }
```

Spring MVC表单标签库

Handler

```java
package com.example.controller;

import com.example.entity.Student;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
@RequestMapping("/tag")
public class TagHandler {
    @GetMapping("/get")
    public ModelAndView get(){
        ModelAndView modelAndView=new ModelAndView("tag");
        Student student=new Student(1,"zhangsan",22);
        modelAndView.addObject(student);
        return modelAndView;
    }
}
```

JSP

```jsp
<%--
  Created by IntelliJ IDEA.
  User: admin
  Date: 2021/2/16
  Time: 15:49
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@page isELIgnored="false" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>学生信息</h1>
<form:form modelAttribute="student">
    学生ID:<form:input path="id"/></br>
    学生姓名:<form:input path="name"/></br>
    学生年龄:<form:input path="age"/></br>
    <input type="submit" value="提交" ></br>
</form:form>
</body>
</html>
```

1、JSP页面导入Spring MVC表单标签库，与导入JSTL语法非常相似，前缀prefix可以自定义，通常定义为form

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

2、将form表单与模型数据进行绑定，通过modelAttribute属性完成绑定，将modelAttribute的值设置为模型数据对应的key值

```
Handler:modelAndView.addObject("student",student);
JSP:<form:form modelAttribute="student">
```

3、form表单完成绑定之后，将模型数据的值取出绑定到不同的标签中，通过设置标签的path属性来完成，将path属性值设置为模型数据对应的属性名即可。

```jsp
    学生ID:<form:input path="id"/></br>
    学生姓名:<form:input path="name"/></br>
    学生年龄:<form:input path="age"/></br>
```

SpringMVC数据校验

- 基于Validator

  实现Validator接口

  ```java
  package com.example.validator;
  
  import com.example.entity.Account;
  import org.springframework.validation.Errors;
  import org.springframework.validation.ValidationUtils;
  import org.springframework.validation.Validator;
  
  public class AccountValidator implements Validator {
      @Override
      public boolean supports(Class<?> aClass) {
          return Account.class.equals(aClass);
      }
  
      //上面返回true执行如下方法
      @Override
      public void validate(Object o, Errors errors) {
          ValidationUtils.rejectIfEmpty(errors,"name",null,"姓名不能为空");
          ValidationUtils.rejectIfEmpty(errors,"password",null,"密码不能为空");
      }
  }
  ```

  Account

  ```java
  package com.example.entity;
  
  import lombok.Data;
  
  @Data
  public class Account {
      private String name;
      private String password;
  }
  
  ```

  Handler

  ```java
  package com.example.controller;
  
  import com.example.entity.Account;
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.validation.BindingErrorProcessor;
  import org.springframework.validation.BindingResult;
  import org.springframework.validation.Errors;
  import org.springframework.validation.annotation.Validated;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  
  @Controller
  @RequestMapping("/validator")
  public class ValidatorHandler {
      @GetMapping("/login")
      public String login(Model model){
          model.addAttribute("account",new Account());
          return "login";
      }
  
      @PostMapping("/login")
      public String login(@Validated Account account, BindingResult bindingResult){
          if(bindingResult.hasErrors()){
              return "login";
          }
          return "index";
      }
  }
  ```

  JSP

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <%@page isELIgnored="false" %>
  <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  <form:form modelAttribute="account" action="/validator/login" method="post">
      用户:<form:input path="name"/><form:errors path="name"/></br>
      密码:<form:input path="password"/><form:errors path="password"/></br>
      <input type="submit" value="提交" ></br>
  </form:form>
  </body>
  </html>
  ```

  springmvc.xml

  ```xml
      <!--基于Validator的配置-->
      <bean id="accountValidator" class="com.example.validator.AccountValidator"></bean>
      <mvc:annotation-driven validator="accountValidator"></mvc:annotation-driven>
  ```

- Annotation JSR-303标准

  使用Annotation JSR-303标准进行验证，需要导入支持这种标准的依赖jar文件，这里我们使用Hibernate Validator。

  pom.xml

  ```xml
   <!--JSR-303-->
      <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>5.3.6.Final</version>
      </dependency>
      <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>2.0.1.Final</version>
      </dependency>
      <dependency>
        <groupId>org.jboss.logging</groupId>
        <artifactId>jboss-logging</artifactId>
        <version>3.3.0.Final</version>
      </dependency>
  ```

  通过注解的方式，直接在实体类中添加相关的验证规则

  ```java
  @Data
  public class Person {
      @NotEmpty(message="用户名不能为空")
      private String username;
      @Size(min=6,max=12,message = "密码6-12位")
      private  String password;
  }
  ```

  ValidatorHandler

  ```java
      @GetMapping("/register")
      public String register(Model model){
          model.addAttribute("person",new Person());
          return "register";
      }
  
      @PostMapping("/register")
      public String register(@Valid Person person,BindingResult bindingResult){
          if(bindingResult.hasErrors()){
              return "register";
          }
          return "index";
      }
  ```

  springmvc.xml

  ```
  <mvc:annotation-driven/>
  ```

  JSP

  ```JSP
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <%@page isELIgnored="false" %>
  <%@taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  <form:form modelAttribute="person" action="/validator/register2" method="post">
      用户:<form:input path="username"/><form:errors path="username"/></br>
      密码:<form:input path="password"/><form:errors path="password"/></br>
      <input type="submit" value="提交" ></br>
  </form:form>
  </body>
  </html>
  ```

  注解校验规则

  @Null:被注解的元素必须为null

  @NotNull:被注解的元素不能为null

  @Min(value):被注解的元素必须是一个数字，其值必须大于等于指定的最小值

  @Max(value):被注解的元素必须是一个数字，其值必须小于等于指定的最大值

  @Email:被注解的元素必须是电子邮箱地址

  @Pattern:被注解的元素必须符合对应的正则表达式

  @Length:被注解的元素的大小必须在指定的范围内

  @NotEmpty:被注解的字符串的值必须非空

  Null和Empty是不同的结果，String str=null,str是null;String str="",str不是null，其值为空

