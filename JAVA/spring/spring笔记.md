### 一、```spring```

优点：轻量级，开源，控制反转(```IOC```)，面向切面(```AOP```)的框架

```java
//框架替代
SSH：Struts2+Spring+Hibernate
		 |	   |	    |
	     ↓	   ↓        ↓
SSM：SpringMVC+Spring+Mybatis
```

#### 1、xml的spring应用

##### 1、1```IOC```推导

原先的服务实现：

1. ```dao层```
2. ```dao层Impl实现类```
3. ```service层```
4. ```service层Impl实现ceng类```
5. ```controller层```(Web)

```service层创建dao的Impl，controller创建service的Impl```

由此每次使用都需要new对象，所以产生spring动态生成Bean对象

##### 1、2```set方法```

在原有的基础上，Service层使用set注入```dao```的接口

```java
//原Service层
public class selectServiceImpl implements selectService {
    @Override
    public void select() {
        //程序员手动new对象，灵活性太差
        selectDao selectDao = new selectDaoImpl();
        selectDao.select();
        System.out.println("service服务接口实现");
    }
}

//使用set注入后，调用serviceImpl的地方直接使用setSelectDao方法手动setdao接口下的实现类，进行动态生成dao的实现类
public class selectServiceImpl implements selectService {
    private selectDao selectDao;
    public void setSelectDao(selectDao selectDao) {
        this.selectDao = selectDao;
    }
    @Override
    public void select() {
        //selectDao selectDao = new selectDaoImpl();
        selectDao.select();
        System.out.println("service服务接口实现");
    }
}
public static void main(String[] args) {
        selectServiceImpl selectService = new selectServiceImpl();
        selectService.setSelectDao(new selectDaoImpl());//动态生成
        selectService.select();
}
```

##### 1、3```两种方法注入（set/有参构造）```

- Set方法

1. 导入依赖(beans,context,core,express,······)

2. 创建需要new的对象(这里以User实体类为例)

   ```java
       private int id;
       private String name;
       private Integer age;
   //（构造/getter/setter/toString方法）······
   ```

3. 创建spring的配置文件xml，申明需要new的对象

   ```xml
   <bean id="user" class="org.study.entity.User">
       <!--给User的属性赋值-->
       <property name="name" value="张三"/>
       <property name="age" value="23"/>
       <!--name：set方法后的名;value：直接赋值给属性;ref：引用生成的实例化对象(id名)-->
   </bean>
   ```

4. 获取spring产生的实例对象

   ```java
   public static void main(String[] args) {
       ApplicationContext context = new ClassPathXmlApplicationContext("spring-cfg.xml");
       //bean设置id，通过id找实体类
       Object user = context.getBean("user");
       System.out.println(user);
   }
   //输出结果：User{id=0, name='张三', age=23}
   ```

   serviceImpl注入daoImpl:

   ```java
   private selectDao selectDao;
   //setter方法
       public void setSelectDao(selectDao selectDao) {
           this.selectDao = selectDao;
       }
       @Override
       public void select() {
           //selectDao selectDao = new selectDaoImpl();
           selectDao.select();
           System.out.println("service服务接口实现");
       }
   ```

   ```xml
   <bean id="123" class="org.study.dao.Impl.selectDaoImpl"/>
       <bean id="selectService" class="org.study.service.Impl.selectServiceImpl">
           <property name="selectDao" ref="123"/>
       </bean>
   ```

- 有参构造注入

  ```java
  private selectDao selectDao;
  //有参构造方法，参数类型selectDao
      public selectServiceImpl(selectDao selectDao) {
          this.selectDao = selectDao;
      }
  ```

  ```xml
  <!--selectDaoImpl类型的Bean-->
  <bean id="123" class="org.study.dao.Impl.selectDaoImpl"/>
      <bean id="selectService" class="org.study.service.Impl.selectServiceImpl">
          <!--有参构造参数注入selectDaoImpl类型的值-->
          <constructor-arg name="selectDao" ref="123"/>
      </bean>
  ```

  调用serviceImpl不报空指针异常，注入成功

  ```java
  ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-cfg.xml");
  		//按照类型查找实例化的Bean
          selectServiceImpl service = context.getBean(selectServiceImpl.class);
          service.select();
  //dao接口的实现
  //service服务接口实现
  ```

##### 1、4```bean作用域```

|    Scope    |                作用域                |
| :---------: | :----------------------------------: |
|  singleton  |      单例：只生成一个实例化对象      |
|  prototype  | 原型：每次获取Bean都新实例化一个Bean |
|   request   |         请求销毁时实例也销毁         |
|   session   |       session销毁时实例也销毁        |
| application |            Web全局都存在             |
|  websocket  |                ······                |

#### 2、注解使用spring

##### 2、1非纯注解使用

1. 添加context约束

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
   ">
   ```

2. 配置注解支持

   ```xml
   <!--注解扫描是针对xml中配置的Bean,spring-MVC中自动加载各种Handler-->
   <context:annotation-config/>
   ```

3. xml中添加Bean

   ```xml
   <bean class="······">······</bean>
   ```


##### 2、2纯注解使用

纯注解不需要set和有参构造，只需要存在属性即可注入。

1. 类上添加注解

   |             注解              |                             作用                             |
   | :---------------------------: | :----------------------------------------------------------: |
   | ```@Component(value="id")```  |                         ```普通类```                         |
   | ```@Repository(value="id")``` |                         ```DAO层```                          |
   |  ```@Service(value="id")```   |                       ```service层```                        |
   |       ```@Autowired```        |                    ```按照类型注入属性```                    |
   | ```@Qualifier(value="id")```  | ```与@Autowired同时使用先按照类型找注入的属性再按照id注入属性``` |
   |  ```@Resource(name="id")```   |     ```相当于@Autowired```+```@Qualifier(value="id")```      |

2. xml中添加context约束,配置扫描注解

   ```xml
   <context:component-scan base-package="org.study">
           <!--包含@···-->
           <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
           <!--排除@Controller-->
           <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
       </context:component-scan>
   ```

#### 3、使用java类申明spring配置

使用注解申明spring配置类

|                            注解                            |                     作用                     |
| :--------------------------------------------------------: | :------------------------------------------: |
|         ```@Configuration(value = "spring-cfg")```         |      作用域：类。申明该类是spring配置类      |
|            ```@ComponentScan({"包1","包2"})```             |         作用域：类。申明注解扫描的包         |
| ```@ComponentScans({@ComponentScan(),@ComponentScan()})``` |            嵌套多个@ComponentScan            |
|                 ```@Bean(value = "id")```                  |            作用域：方法。申明Bean            |
|        ```@Import(value = {*.class1, *.class2})```         | 作用域：类。团队开发，导入其他spring的配置类 |

###### 使用时```new AnnotationConfigApplicationContext(配置类的.class);```

### 二、**Spring-mvc**

概述：基于java的实现MVC设计模型的请求驱动类型的轻量级Web框架。

MVC概述：

- Model:数据封装和业务逻辑处理
- View:数据展示
- Controller:控制器，分发指派工作

###### spring-MVC优点:使用一套注解使一个类成为处理请求的控制器，无需实现任何接口,还支持RESTful编程风格的请求。

--->原先的实现HttpServlet接口的类(Web层)被Controller替代<---

spring-mvc开发过程：



spring-MVC执行流程：



前端控制器：根据请求调度组件

处理器映射器:解析请求，确认找哪个资源，返回一串执行链

处理器适配器:调度要执行的Controller，返回ModelAndView

视图解析器：解析ModelAndView，返回View对象



#### **注解和配置**

|          注解           |                          作用                          |
| :---------------------: | :----------------------------------------------------: |
|  ```@RequestMapping```  |         请求映射,把请求的虚拟地址映射到方法上          |
|   ```@ResponseBody```   | 数据回写时不返回视图(**.jsp···),直接在http响应体中返回 |
|   ```@RequestBody```    |                 请求参数是json的请求体                 |
|   ```@RequestParam```   |                  前后端的请求参数绑定                  |
| **```@PathVariable```** |      **RESTful风格**请求使用占位符时进行参数绑定       |
|  ```@RequestHeader```   |                  参数绑定，获取请求头                  |
|   ```@CookieValue```    |                  参数绑定，获取Cookie                  |

对`@PathVariable`，`@RequestParam`，`@RequestBody`三者的比较

|       注解        | 支持的类型 |    支持的请求类型     |        支持的Content-Type        |  请求示例  |
| :---------------: | :--------: | :-------------------: | :------------------------------: | :--------: |
| **@PathVariable** |    url     |          get          |               所有               | /test/{id} |
|   @RequestParam   |    url     |          get          |               所有               | /test?id=1 |
|   @RequestParam   |    Body    | POST/PUT/DELETE/PATCH | form-data或x-www.form-urlencoded |    id:1    |
|   @RequestBody    |    Body    | POST/PUT/DELETE/PATCH |               json               |  {"id":1}  |



##### 注解解释

- ```@RequestMapping```

  value:请求虚拟地址

  method:请求方法(枚举类型:RequestMethod.GET)

  params:限定请求参数条件,支持简单表达式(如:必须携带name，不带参数则请求失败)

- ```@ResponseBody```

  数据回写时不返回视图

- ```@RequestBody```

  作用于参数前，能自动把提交的很多个单独的entity封装为对应的List<E>

- ```@RequestParam```

  name/value:前端传来的参数名。

  required:是否必须有这个参数

  defaultValue:默认参数值，如果没有这个参数则使用默认参数

  ```java
  //前端传来name与后端的userName做绑定
  @RequestMapping("/test")
      public ModelAndView test(@RequestParam(value = "name") String userName, ModelAndView modelAndView){
          return modelAndView;
      }
  ```

- ```PathVariable```

  name/value:

  required:是否必须有这个参数

  **RESTful**风格时:获取请求参数(get)

  ```java
  @RequestMapping("/test2/{name}")
  @ResponseBody
      public void test2(@PathVariable("name") String userName, ModelAndView modelAndView){
  //请求URL中最后是/abc所以abc赋值给占位符{name},方法参数中用注解进行参数绑定
          System.out.println(userName);
      }
  //请求url:localhost:8080/test2/abc
  ```

- ```@RequestHeader```

  name/value:指定请求头是谁(如**User-Agent**)

  required:是否必须有这个参数

  ```java
  @RequestMapping("/test3")
  @ResponseBody
      public void test2(@RequestHeader("User-Agent") String user_agent, ModelAndView modelAndView){
          //打印出浏览器的User-Agent
          System.out.println(user_agent);
      }
  ```

- ```@CookieValue```

  name/value:指定

  required:是否必须有这个参数

  defaultValue:

  ```java
  @RequestMapping("/test4")
  @ResponseBody
      public void test3(@CookieValue(value="JSESSIONID") String Cookie, ModelAndView modelAndView){
          //获取JSESSIONID=后啊免得值
          System.out.println(Cookie);
      }
  ```

  



```xml
<!--加载注解驱动及各种Handler,并使用自定义类型转换器-->
<mvc:annotation-driven conversion-service="conversionService"/>
<bean class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean>自定义类型转换类</bean>
            </list>
        </property>
    </bean>
<!--使用tomcat的servlet处理器开放资源访问-->
<mvc:default-servlet-handler />
<!--mvc拦截器们-->
<mvc:interceptors>
        <mvc:interceptor>
            <!--path是拦截哪些请求路径,/**便是-->
            <mvc:mapping path="/user"/>
            <bean>实现了HandlerInterceptor的类</bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

###### RESTful风格：请求方法及含义：get查询post添加put修改delete删除

###### 拦截器:```Interceptor```具体使用

1. 自定义拦截器

   ```java
   package com.interceptor;
   
   import org.springframework.web.servlet.HandlerInterceptor;
   import org.springframework.web.servlet.ModelAndView;
   
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   
   public class MyInterceptor implements HandlerInterceptor {
       
       //controller层方法执行前
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           System.out.println("拦截器preHandle方法");
           return false;//不放行,可以重定向，转发
       }
       //controller层方法执行后，视图返回前
       @Override
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
           System.out.println("拦截器postHandle方法");
       }
       //controller层方法执行完毕后
       @Override
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
           System.out.println("拦截器afterCompletion方法");
       }
   }
   ```

2. mvc.xml配置文件

   ```xml
       <!--拦截器，可配置多个拦截器，按配置文件顺序进行拦截-->
       <mvc:interceptors>
           <mvc:interceptor>
               <mvc:mapping path="/**"/>
               <bean class="com.interceptor.MyInterceptor"/>
           </mvc:interceptor>
       </mvc:interceptors>
   ```




###### 文件上传

1. 依赖包

   ```java
       <!--文件上传-->
       <dependency>
         <groupId>commons-fileupload</groupId>
         <artifactId>commons-fileupload</artifactId>
         <version>1.4</version>
       </dependency>
       <dependency>
         <groupId>commons-io</groupId>
         <artifactId>commons-io</artifactId>
         <version>2.11.0</version>
       </dependency>
   ```

2. jsp上传，交给controller层

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>upload</title>
   </head>
   <body>
   <form action="${pageContext.request.contextPath}/upload/files" method="post" enctype="multipart/form-data">
       名<input name="name" type="text"><br/>
       文件<input name="file" type="file"><br/>
       <input type="submit" value="提交">
       <input type="reset" value="重置">
   </form>
   </body>
   </html>
   ```

3. controller层

   ```java
   package com.controller;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.multipart.MultipartFile;
   import java.io.File;
   import java.io.IOException;
   
   @Controller(value = "uploadFiles")
   @RequestMapping(value = "/upload")
   public class uploadController {
       @RequestMapping(value = "/files")
       @ResponseBody
       public void upload(String name,MultipartFile file) throws IOException {
           //获取文件名
           String fileName = file.getOriginalFilename();
           System.out.println(fileName);//新建文本文档.txt
           System.out.println(name);//111
           System.out.println(file);//MultipartFile[field="file", filename=新建文本文档.txt, contentType=text/plain, size=16]
           file.transferTo(new File("D:\\文件\\Desktop\\新建文件夹\\"+fileName));
       }
   }
   ```
   
4. mvc配置

   ```xml
   <!--文件上传解析器-->
       <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
           <property name="defaultEncoding" value="UTF-8"/>
           <property name="maxUploadSizePerFile" value="5242800"/><!--单个文件大小5mb-->
           <property name="maxUploadSize" value="524800"/><!--上传总文件大小-->
       </bean>
   ```

### 异常处理

- 使用spring的简单异常处理  ```SimpleMappingExceptionResolver```  **配置spring-mvc.xml就可以自动使用**

  ```xml
      <!--异常处理-->
      <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
          <!--下面的异常不匹配，最后默认异常跳转到error.jsp-->
          <property name="defaultErrorView" value="error"/>
          <!--每个异常跳转到error.jsp-->
          <property name="exceptionMappings">
              <map>
                  <entry key="java.lang.ClassCastException" value="error"/>
                  <entry key="java.io.FileNotFoundException" value="error"/>
              </map>
          </property>
  ```

- 自定义异常处理

  1. 创建异常处理器类实现```HandlerExceptionResolver```

     1.1处理类

     ```java
     package com.Exception;
     import org.springframework.web.servlet.HandlerExceptionResolver;
     import org.springframework.web.servlet.ModelAndView;
     import javax.servlet.http.HttpServletRequest;
     import javax.servlet.http.HttpServletResponse;
     import java.io.FileNotFoundException;
     
     public class myException implements HandlerExceptionResolver {
         @Override
         public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
             ModelAndView modelAndView = new ModelAndView();
             if (ex instanceof ClassCastException) {
                 modelAndView.addObject("info","类转换异常信息");
             }else if (ex instanceof FileNotFoundException) {
                 modelAndView.addObject("info","文件找不到异常");
             }
             modelAndView.setViewName("error");//有错则找error页面
             return modelAndView;
         }
     }
     ```
  
     1.2```spring-mvc.xml```配置，添加实现了HandlerExceptionResolver的类
  
     ```xml
     <!--ti自定义异常处理-->
         <bean class="com.Exception.myException"/>
     ```



