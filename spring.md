#### Spring框架两大核心机制

- IoC（控制反转）:创建各种对象/DI（依赖注入）：将各个对象关联起来，实现在一个对象中使用另一个对象
- AOP面向切面编程：将不同业务中相同的代码，抽象成一个对象，进行集中管理

Spring是一个企业级开发框架，是软件设计层面的框架，优势在于可以将应用程序分层，开发者可以自主选择组件。

MVC:Struts、SpringMVC

ORMapping(持久层）:Hibernate、MyBatis、Spring Data 

#### 控制反转

概念：传统的程序开发中，对象是由调用者主动new出来的；spring框架中，IoC完成容器的创建，再推送给调用者

**如何使用IoC**:

- 创建Maven工程，pom.xml添加依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springioc</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.11.RELEASE</version>
        </dependency>
        <!--自动生成set、get、toString方法-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.14</version>
        </dependency>
    </dependencies>
</project>
```

- 通过IoC创建对象，在配置文件中添加需要管理的对象，xml格式的配置文件，文件名可自定义。放在resources下面，java代码都放到java包下。spring.xml配置bean文件如下：

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:p="http://www.springframework.org/schema/p"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
      					http://www.springframework.org/schema/beans/spring-beans.xsd
      					http://www.springframework.org/schema/context
      					http://www.springframework.org/schema/context/spring-context.xsd">
      <bean id="student" class="entity.Student">
          <property name="id" value="1"></property>
          <property name="age" value="22"></property>
          <property name="name" value="zhangsan"></property>
      </bean>
  </beans>
  ```

- 从IoC中获取对象，通过id获取：

  ```
  public class Test {
      public static void main(String[] args){
      //加载配置文件
          ApplicationContext applicationContext=new ClassPathXmlApplicationContext("spring.xml");
          Student student= (Student) applicationContext.getBean("student");
          System.out.println(student);
      }
  }
  ```

  ##### 配置文件

- 通过bean标签来完成对象的管理

  - id：对象名

  - class：对象的全类名。（所有交给IoC容器管理的类必须有无参构造函数，因为Spring底层是通过反射机制来创建对象，调用的是无参构造）@AllArgsConstuctor注解代表所有的有参构造，@NoArgsConstructor注解代表无参构造，两个都不写默认是有无参的

- 对象的成员变量通过property标签完成赋值

  - name：成员变量名

  - value：成员变量的值(基本数据类型，String可以直接赋值，如果是其他引用类型，不能通过value赋值)

  - ref：将IoC中的另外一个bean赋给当前的成员变量（DI)

```xml
    <bean id="student" class="entity.Student">
        <property name="id" value="1"></property>
        <property name="age" value="22"></property>
        <property name="name" value="zhangsan"></property>
        <property name="address" ref="address"></property>
    </bean>
    <bean id="address" class="entity.Address">
        <property name="id" value="1"></property>
        <property name="name" value="科技路"></property>
    </bean>
```

##### IoC底层原理

- 读取配置文件、解析xml
- 通过反射机制实例化配置文件中所配置的所有bean(./代表当前工程)

```java
package ioc;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class ClassPathXmlApplicationContext implements ApplicationContext{
    Map<String,Object> ioc=new HashMap<String,Object>();
    public ClassPathXmlApplicationContext(String path){
        try{
            SAXReader reader=new SAXReader();
            Document document=reader.read(path);
            Element root=document.getRootElement();
            Iterator<Element> iterator=root.elementIterator();
            while(iterator.hasNext()){
                Element element=iterator.next();
                String id=element.attributeValue("id");
                String className=element.attributeValue("class");
                Class clazz=Class.forName(className);
                //获取无参构造函数，创建目标对象
                Constructor constructor=clazz.getConstructor();
                Object object=constructor.newInstance();
                //给目标对象赋值,迭代每个bean下的property属性
                Iterator<Element> beanIterator=element.elementIterator();
                while(beanIterator.hasNext()){
                    Element propertyElement=beanIterator.next();
                    String name=propertyElement.attributeValue("name");
                    String valueStr=propertyElement.attributeValue("value");
                    String ref=propertyElement.attributeValue("ref");
                    if(ref==null) {
                        String methodName = "set" + name.substring(0, 1).toUpperCase() + name.substring(1);
                        //获取某个公共字段
                        Field field = clazz.getDeclaredField(name);
                        //得到set方法，第一个参数为方法名，第二个为参数类型
                        Method method = clazz.getDeclaredMethod(methodName, field.getType());
                        //根据成员变量的数值类型，将值进行转换
                        Object value = null;
                        if (field.getType().getName() == "int") {
                            value = Integer.parseInt(valueStr);
                        }
                        if (field.getType().getName() == "java.lang.String") {
                            value = valueStr;
                        }
                        if (field.getType().getName() == "long") {
                            value = Long.parseLong(valueStr);
                        }
                        //赋值，第一个参数为对象，第二个为值
                        method.invoke(object, value);
                    }
                }
                ioc.put(id,object);
            }
//            Object obj1=ioc.get("address");
//            Object obj2=ioc.get("student");

        }catch (DocumentException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
    public Object getBean(String id) {
        return ioc.get(id);
    }
}
```

###通过运行时类获取bean

```java
ApplicationContext applicationContext=new ClassPathXmlApplicationContext("spring.xml");
        //Student student= (Student) applicationContext.getBean("student");
        Student student= (Student) applicationContext.getBean(Student.class);
        System.out.println(student);
```

这种方式存在一个问题，配置文件中一个数据类型的对象只能有一个实例，否则会抛出异常，因为没有唯一的bean。

###通过有参构造来创建bean

- 在实体类中创建对应的有参构造函数

- 配置文件

  ```xml
  <bean id="student3" class="entity.Student">
          <constructor-arg name="id" value="3"></constructor-arg>
          <constructor-arg name="age" value="18"></constructor-arg>
          <constructor-arg name="name" value="小明"></constructor-arg>
          <constructor-arg name="address" ref="address"></constructor-arg>
  </bean>
  或者
  <bean id="student3" class="entity.Student">
          <constructor-arg index="0" value="3"></constructor-arg>
          <constructor-arg index="1" value="18"></constructor-arg>
          <constructor-arg index="2" value="小明"></constructor-arg>
          <constructor-arg index="3" ref="address"></constructor-arg>
  </bean>
  ```

  ###给bean注入集合
  
  ```java
      <bean id="student" class="entity.Student">
          <property name="id" value="1"></property>
          <property name="age" value="22"></property>
          <property name="name" value="zhangsan"></property>
          <property name="addresses">
              <list>
                  <ref bean="address"></ref>
                  <ref bean="address2"></ref>
              </list>
          </property>
      </bean>
      <bean id="address" class="entity.Address">
          <property name="id" value="1"></property>
          <property name="name" value="科技路"></property>
      </bean>
  
      <bean id="address2" class="entity.Address">
          <property name="id" value="2"></property>
          <property name="name" value="高新区"></property>
      </bean>
  ```
  
  

##### scope作用域

spring管理的bean是根据scope来生成的，表示bean的作用域，共有四种。默认值是singleton

- singleton：单例，表示通过IoC容器获取的bean是唯一的
- prototype：原型，表示通过IoC容器获取的bean是不同的
- request:请求，表示在一次HTTP请求内有效
- session:回话，表示在一个用户会话内有效

request和Session只适用于Web项目，大多数情况下，使用单例和原型较多

prototype模式当业务代码获取IoC中的bean时(调用getBean（）方法），Spring才去调用无参构造创建对应的bean（用的时候创建）。

singleton模式无论业务代码是否获取IoC容器中的bean，Spring在加载spring.xml时就会调用无参构造方法创建bean。

##### Spring的继承

与java中的继承不同，java是类层面的继承，子类可以继承父类的内部结构信息，Spring是对象层面的继承，子对象可以继承父对象的属性值。

```java
    <bean id="student" class="entity.Student">
        <property name="id" value="1"></property>
        <property name="age" value="22"></property>
        <property name="name" value="zhangsan"></property>
        <property name="addresses">
            <list>
                <ref bean="address"></ref>
                <ref bean="address2"></ref>
            </list>
        </property>
    </bean>

    <bean id="address" class="entity.Address">
        <property name="id" value="1"></property>
        <property name="name" value="科技路"></property>
    </bean>

    <bean id="address2" class="entity.Address">
        <property name="id" value="2"></property>
        <property name="name" value="高新区"></property>
    </bean>

    <bean id="stu" class="entity.Student"  parent="student">
        <property name="name" value="李四"></property>
    </bean>
```

Spring的继承关注点在于具体的对象，而不在于类，即不同的两个类的实例化对象（bean）可以完成继承，前提是子对象必须包含父对象的所有属性，同时可以在此基础上添加其他的属性。

##### Spring的依赖

与继承类似，依赖也是描述bean和bean之间的一种关系，配置依赖之后，被依赖的bean一定先创建，再创建依赖的bean，A依赖于B，先创建B，再创建A。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    					http://www.springframework.org/schema/beans/spring-beans.xsd
    					http://www.springframework.org/schema/context
    					http://www.springframework.org/schema/context/spring-context.xsd">
    <bean id="student" class="entity.Student" depends-on="User"></bean>

    <bean id="User" class="entity.User"></bean>

</beans>
```

##### Spring的p命名空间

p命名空间是对Ioc/DI的简化，使用p命名空间可以更加方便的完成bean的配置以及bean之间的依赖注入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    					http://www.springframework.org/schema/beans/spring-beans.xsd
    					http://www.springframework.org/schema/context
    					http://www.springframework.org/schema/context/spring-context.xsd">
    <bean id="student" class="entity.Student" p:id="1" p:name="zhangsan" p:age="12" p:address-ref="address"></bean>

    <bean id="address" class="entity.Address" p:id="1" p:name="科技路"></bean>
</beans>
```

##### Spring的工厂方法

Ioc通过工厂模式创建bean的方式有两种：

- 静态工厂方法
- 实例工厂方法

​    静态工厂方法

```java
public class Test4 {
    public static void main(String[] args){
        ApplicationContext applicationContext=new ClassPathXmlApplicationContext("./src/main/resources/spring-factory.xml");
        Car car= (Car) applicationContext.getBean("car");
        System.out.println(car);
    }
}
```

```java
public class StaticCarFactory {
     private static Map<Integer, Car> carMap;
     static{
         carMap=new HashMap<Integer, Car>();
         carMap.put(1,new Car(1,"宝马"));
         carMap.put(2,new Car(2,"奔驰"));
     }
     public static Car getCar(Integer id){
         return carMap.get(id);
     }
}
```

```xml
    <!--配置静态工厂创建 Car-->
    <bean id="car" class="factory.StaticCarFactory" factory-method="getCar">
        <constructor-arg value="1"></constructor-arg>
    </bean>
```

   实例工厂方法

```java
public class InstanceCarFactory {
    private Map<Integer, Car> carMap;
    public InstanceCarFactory(){
        carMap=new HashMap<Integer, Car>();
        carMap.put(1,new Car(1,"宝马"));
        carMap.put(2,new Car(2,"奔驰"));
    }
    public Car getCar(Integer id){
        return carMap.get(id);
    }
}

```

```xml
    <!--配置实例工厂bean-->
    <bean id="carFactory" class="factory.InstanceCarFactory">
    </bean>

    <!--通过实例工厂创建 Car-->
    <bean id="car2" factory-bean="carFactory" factory-method="getCar">
        <constructor-arg value="1"></constructor-arg>
    </bean>
```

##### IoC自动装载（Autowire）

IoC负责创建对象，DI负责完成对象的依赖注入，通过配置property标签的ref属性来完成，同时Spring提供了另外一种更加简便的依赖注入方式，自动装载，不需要手动配置property，IoC容器会自动选择bean完成注入。

自动装载有两种方式：

- byName：通过属性名自动装载

  ```xml
      <bean id="car" class="entity.Car">
          <property name="id" value="1"></property>
          <property name="name" value="宝马"></property>
      </bean>
  
      <bean id="person" class="entity.Person" autowire="byName">
          <property name="id" value="11"></property>
          <property name="name" value="张三"></property>
      </bean>
  ```

- byType：通过属性的数据类型自动装载

​       注意，如果说同时存在两个及以上的符合条件的bean时，自动装载会抛出异常。

##### AOP：面向切面编程（将散布在系统中的公共功能集中解决）

AOP优点：降低模块之间的耦合度、使系统容易扩展、更好的代码复用、非业务代码更加集中，不分散，便于统一管理、业务代码更加简洁纯粹，不惨杂其他代码的影响AOP

AOP是对面向对象编程的一个补充，在运行时，动态的将代码切入到指定方法，指定位置上的编程，将不同方法的同一个位置抽象成一个切面对象，对该切面对象进行编程

##### 如何使用？

创建maven工程，pom.xml中添加

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.3.7.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.7.RELEASE</version>
</dependency>
```

- 创建一个计算器接口Cal，定义4个方法

  ```java
  public interface Cal {
      public int add(int num1,int num2);
      public int sub(int num1,int num2);
      public int mul(int num1,int num2);
      public int div(int num1,int num2);
  }
  ```

- 创建接口的实现类CalImpl

  ```java
  public class CalImpl implements Cal {
      public int add(int num1, int num2) {
          System.out.println("add方法的参数是["+num1+","+num2+"]");
          int result=num1+num2;
          System.out.println("add方法的结果是"+result);
          return result;
      }
  
      public int sub(int num1, int num2) {
          System.out.println("sub方法的参数是["+num1+","+num2+"]");
          int result=num1-num2;
          System.out.println("sub方法的结果是"+result);
          return result;
      }
  
      public int mul(int num1, int num2) {
          System.out.println("mul方法的参数是["+num1+","+num2+"]");
          int result=num1*num2;
          System.out.println("mul方法的结果是"+result);
          return result;
      }
  
      public int div(int num1, int num2) {
          System.out.println("div方法的参数是["+num1+","+num2+"]");
          int result=num1/num2;
          System.out.println("div方法的结果是"+result);
          return result;
      }
  }
  ```

  上述代码中，日志信息和业务逻辑的耦合性很高，不利于系统的维护，使用AOP可以进行优化，如何来实现AOP？使用动态代理的方式来实现。

  给业务代码找一个代理，打印日志信息的工作交给代理来做，这样的话业务代码只需关注自身的业务即可