





##### ```springboot```自动配置



```Condition```：创建bean的时的条件判断。

1. 实现```Condition```接口，定义判断方法

2. ```spring```的配置类(```@Configuration```)中的配置Bean方法(@Bean)后添加判断注解```@Conditional```(实现```Condition```接口类的字节码文件)

3. ```sprintboot```提供的条件注解(底层依赖于@Conditional)

   - ```@ConditionalOn···```

     ```java
     @ConditionalOnClass
     @ConditionalOnExpression
     ····
     ```

     























