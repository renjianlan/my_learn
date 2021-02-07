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



