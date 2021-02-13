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
      public DateConverter(String pattern){
          this.pattern=pattern;
      }
  
      @Override
      public Date convert(String s) {
         SimpleDateFormat simpleDateFormat=new SimpleDateFormat(this.pattern);
         Date date=null;
         try {
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

  

