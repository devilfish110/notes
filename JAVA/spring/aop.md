###### 代理

1. 静态代理：有一个.java的类文件描述代理模式
2. 动态代理：没有类文件，在内存中动态生成代理类

### 一、```JDK```动态代理(基于接口实现动态代理：兄弟)

1. **接口**

   ```java
   package com.JDKProxy;
   public interface TargetInterface {
       public abstract void method();
   }
   ```
   
2. **实现类**(目标类)

   ```java
   package com.JDKProxy;
   public class Target implements TargetInterface{
       @Override
       public void method() {
           System.out.println("目标对象方法");
       }
   }
   ```
   
3. **增强类**

   ```java
   package com.JDKProxy;
   public class Advice {
       //前置增强
       public void before(){
           System.out.println("前置增强");
       }
       //后置增强
       public void after(){
           System.out.println("后置增强");
       }
   }
   ```
   
4. 测试

   ```java
   package com.test;
   
   import com.JDKProxy.Advice;
   import com.JDKProxy.TargetInterface;
   import com.JDKProxy.Target;
   
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   
   public class JDKProxyTest {
       public static void main(String[] args) {
           //创建目标对象TargetInterfaceImpl
           Target target = new Target();
           //创建增强对象Advice
           Advice advice = new Advice();
           //返回值:基于接口动态生成的代理对象
           TargetInterface poxy = (TargetInterface) Proxy.newProxyInstance(
                   //目标对象.class加载器
                   target.getClass().getClassLoader(),
                   //目标对象的接口字节码(对象数组)
                   target.getClass().getInterfaces(),
                   new InvocationHandler() {
                       //调用代理对象的任何方法都会执行invoke()方法
                       /**
                        * @param proxy 代理对象
                        * @param method 代理对象调用的方法被封装的对象
                        * @param args 代理对象调用方法时，传递的实际参数
                        */
                       @Override
                       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                           //前置增强
                           advice.before();
                           //执行目标方法
                           Object invoke = method.invoke(target, args);
                           //后置增强
                           advice.after();
                           return invoke;
                       }
                   }
           );
           //调用代理对象方法
           poxy.method();
       }
   }
   ```

### 二、```cglib```动态代理(基于父类实现动态代理：儿子)

原理：找个比父类更强大的儿子

1. 导入依赖(spring5集成)

2. 目标类

   ```java
   package com.cglibProxy;
   
   public class Target {
       public void method(){
           System.out.println("目标方法");
       }
   }
   ```

3. 增强类

   ```java
   package com.cglibProxy;
   
   public class Advice {
       //前置增强
       public void before(){
           System.out.println("前置增强");
       }
       //后置增强
       public void after(){
           System.out.println("后置增强");
       }
   }
   ```

4. 测试类

   ```java
   package com.test;
   
   import com.cglibProxy.Advice;
   import com.cglibProxy.Target;
   import org.springframework.cglib.proxy.Enhancer;
   import org.springframework.cglib.proxy.MethodInterceptor;
   import org.springframework.cglib.proxy.MethodProxy;
   
   import java.lang.reflect.Method;
   
   public class cglibProxyTest {
       public static void main(String[] args) {
           //创建目标对象TargetInterfaceImpl
           Target target = new Target();
           //创建增强对象Advice
           Advice advice = new Advice();
           //1.创建增强器
           Enhancer enhancer = new Enhancer();
           //2.设置增强器父类(目标对象.classzi'ji)
           enhancer.setSuperclass(Target.class);
           //3.设置回调
           enhancer.setCallback(new MethodInterceptor() {
               @Override
               public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                   //前置增强
                   advice.before();
                   //执行目标方法
                   Object invoke = method.invoke(target, args);
                   //后置增强
                   advice.after();
                   return invoke;
               }
           });
           //4.创建代理对象
           Target proxy = (Target) enhancer.create();
           proxy.method();
       }
   }
   ```

###### 两个代理执行结果一样

###### ```aop```术语：

- ```target```(目标对象)：被代理的对象
- ```Proxy```(代理)：生成的代理类
- ```Joinpoint```(连接点)：被拦截到的点。```spring```中指方法(可以被增强的方法)
- **```Pointcut```(切入点/切点)**：对哪些```Joinpoint```进行拦截的定义(实际被增强的方法)
- **```Advice```(通知/增强)**：拦截到的```Joinpoint```之后所要做的事(增强方法/增强逻辑)
- **Aspect(切面)**：切入点和增强的结合
- Weaving(织入)：把增强应用到目标对象来创建代理对象的过程。```spring```采用动态代理织入，```Aspect```采用编译期织入和类装载期织入

### spring集成```aop```代理

- **XML开发**

1. 导依赖

   ```xml
       <!--第三方aop依赖(注解)-->
   	<dependency>
         <groupId>org.aspectj</groupId>
         <artifactId>aspectjweaver</artifactId>
           <version>1.9.9.1</version>
       </dependency>
   ```

2. 接口及其实现类target

   ```java
   package com.aop;
   
   import com.entity.User;
   
   public interface UserInterface {
       public abstract User getUsername(Integer id);
   }
   ```

   ```java
   package com.aop;
   
   import com.entity.User;
   
   public class UserService implements UserInterface{
       @Override
       public User getUsername(Integer id) {
           System.out.println("用户id为："+id);
           User user = new User();
           user.setId(id);
           user.setLoginName("root");
           user.setPassword("root");
           return user;
       }
   }
   ```

3. ```Advice```类：增强类(增强逻辑)

   ```java
   package com.aop;
   
   public class Advice {
       public void abc(){
           System.out.println("前置增强·····");
       }
       public void after(){
           System.out.println("后置增强·······");
       }
   }
   ```

4. ```spring.xml```配置

   ```xml
       <!--aop-->
       <bean id="service" class="com.aop.UserService"/>
       <bean id="advice" class="com.aop.Advice"/>
   	<!--proxy-target-class使用基于类创建代理-->
       <aop:config proxy-target-class="true">
           <!--抽取切点表达式-->
           <aop:pointcut id="111" expression="execution(* com.aop.UserService.*(..))"/>
           <!--Aspect：把增强(advice)逻辑切入到切点(execution(切点表达式))-->
           <aop:aspect ref="advice">
               <!--把method方法增强给后面的pointcut方法-->
               <!--前置通知-->
               <aop:before method="abc" pointcut="execution(public com.entity.User com.aop.UserService.getUsername(Integer))"/>
               <!--后置通知-->
               <aop:after-returning method="after" pointcut="execution(com.entity.User com.aop.UserService.getUsername(Integer))"/>
               <!--环绕通知-->
               <aop:around method="abc" pointcut-ref="111"/>
               <!--异常抛出通知-->
               <aop:after-throwing method="abc" pointcut-ref="111"/>
               <!--最终通知-->
               <aop:after method="after" pointcut-ref="111"/>
           </aop:aspect>
       </aop:config>
   ```

   **切点表达式**

   **```execution([修饰符] 返回值类型 包名.类名.方法名(参数))```**

   注：

   1. 切点表达式格式：使用切点的全限定名 

   ​			如：切点(方法)为```public User getUsername(Integer id)```   则切点表达式为：```execution(public User所在包位置 切点类所在包位置.getUsername(Integet)```

   ​	2.参数可以使用..表示任意个数，任意类型的参数列表

- **注解开发**

  1. 配置```@Component```

     ```java
     //两个类配置bean
     @Component("userService")
     @Component("advice")
     ```

  2. 在切面类中配置**织入**关系

     ```java
     //在zeng'q下添加
     @Aspect//告诉spring这个类是切面类
     //在切面类的方法前配置注解
     //@Before() @AfterReturning() @AfterThrowing() @After()
     //切面表达式注解@Pointcut()
     @Before(value = "execution(* com.aop.UserService.*(..))")
     ```

  3. ```spring.xml```中开启组件扫描和```aop```代理自动代理

     ```xml
     <!--组件扫描com下的除@Controller注解-->
     <context:component-scan base-package="com">
             <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
         </context:component-scan>
     <!--aop自动代理-->
     <aop:aspectj-autoproxy proxy-target-class="true"/>
     ```

     

     ```java
     	/*抽取切面表达式 */    
     	//@AfterReturning(value = "Advice.a()")或@AfterReturning(value = "a()")
     	@AfterReturning(value = "Advice.a()")
         public void after(){
             System.out.println("后置增强·······");
         }
     	//空方法提取切面表达式
         @Pointcut("execution(* com.aop.UserService.*(..))")
         public void a(){}
     ```

     

mybatis/aop集成

1. 平台事务管理器
