#### ```Spring事务处理```

概念：              commit()提交事务rollback()回滚事务

- 编程式事务控制：

  1. ```PlatformTransactionManager```    平台事务管理器(接口类型)

     |                             方法                             |        说明        |
     | :----------------------------------------------------------: | :----------------: |
     | ```TransationStatus getTransaction(TransationDefination defination)``` | 获取事务的状态信息 |
     |          ```void commit(TransationStatus status)```          |      提交事务      |
     |         ```void rollback(TransationStatus status)```         |      回滚事务      |

     注：不同的```Dao```层有不同的实现类。

     ​		```jdbc```或```mybatis```：```org.springframework.jdbc.datasource.DatasourceTransactionManager```

     ​		```Hibernate```：```org.springframework.orm.hibernate5.HibernateTransactionManager```

  2. ```TransactionDefinition```     事务的定义对象(封装了控制事务参数)

     |                方法                |        说明        |
     | :--------------------------------: | :----------------: |
     |   ```int getIsolationLevel()```    | 获得事务的隔离级别 |
     | ```int getPropogationBehavior()``` | 获得事务的传播行为 |
     |       ```int getTimeout()```       |    获得超时时间    |
     |     ```boolean isReadOnly()```     |      是否只读      |

     1. 事物的隔离级别

        解决事务并发产生的问题(脏读，不可重复读，虚读)

        - ```ISOLATION_DEFAULT```     默认
        - ```ISOLATION_READ_UNCOMMITTED ```     读不可提交的
        - ```ISOLATION_READ_COMMITTED```     读提交的
        - ```ISOLATION_REPEATABLE_READ ```     
        - ```ISOLATION_SERIALIZABLE```     

     2. 事务的传播行为

        概念：解决业务方法调用业务方法时事务统一性问题

        - ```REQUIRED```：如果当前没有事务,则新建事务,如果有事务,则加入事务(默认)

          A业务方法  ===>  B业务方法  B看A当前有没有事务,有事务则加入事务,没有事务则创建事务

        - ```SUPPPORTS```：支持当前事务,如果没有事务,以非事务方式运行

          A业务方法  ===>  B业务方法  B看A当前有没有事务,有事务则支持事务,没有事务则以非事务方式运行

        - 超时时间：默认值-1，没有时间限制，以s为单位设置，解决雪崩问题

        - 是否只读：建议查询时设置只读

        - ···············

  3. ```TransactionStatus```     事务状态

     |               方法               |      说明      |
     | :------------------------------: | :------------: |
     |   ```boolean hasSavepoint()```   | 是否存储回滚点 |
     |    ```boolean isCmpleted()```    |  事务是否完成  |
     | ```boolean isNewTransaction()``` |  是否是新事务  |
     |  ```boolean isRollbackOnly()```  |  事务是否回滚  |

     

  **```TransactionStatus```=```PlatformTransactionManager```+```TransactionDefinition```**

- 声明式事务控制：采用声名的方式处理事务

  1. 基于XML声名事务控制

     1. 导入```spring```基本依赖及事务依和**```aop```**依赖

        ```xml
            <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-tx</artifactId>
              <version>5.3.19</version>
            </dependency>
            <!--第三方aop依赖-->
        	<dependency>
              <groupId>org.aspectj</groupId>
              <artifactId>aspectjweaver</artifactId>
                <version>1.9.9.1</version>
            </dependency>
        ```

     2. ```spring-context.xml```(Service层使用事务)配置

        1. 添加约束

           ```xml
           xmlns:tx="http://www.springframework.org/schema/tx"
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
           ```

        2. 配置数据源

           ```xml
               <!--加载jdbc资源-->
               <context:property-placeholder location="classpath:jdbc.properties"/>
               <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
                   <property name="driverClassName" value="${jdbc.driver}"/>
                   <property name="url" value="${jdbc.url}"/>
                   <property name="username" value="${jdbc.username}"/>
                   <property name="password" value="${jdbc.password}"/>
               </bean>
           ```

        3. **<u>配置平台事务管理器</u>**

           ```xml
           <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
                   <!--注入dataSource,从dataSource里获取conn-->
                   <property name="dataSource" ref="dataSource"/>
               </bean>
           ```

        4. **<u>配置事务的增强</u>**

           ```xml
           <!--通知  事务的增强-->
               <tx:advice id="txAdvice" transaction-manager="transactionManager">
                   <!--设置事务的属性信息(配置事务参数:TransactionDefinition层)-->
                   <tx:attributes>
                       <!--配置切点的方法属性参数-->
                       <tx:method name="*" read-only="false"/>
                       <tx:method name="方法名" read-only="true" propagation="REQUIRED"/>
                   </tx:attributes>
               </tx:advice>
           ```

        5. **<u>事务织入</u>**

           ```xml
               <!--事务aop织入-->
               <aop:config proxy-target-class="true">
                   <!--把增强的txAdvice用于需要增强的方法-->
                   <aop:advisor advice-ref="txAdvice" pointcut="切入表达式"/>
               </aop:config>
           ```

  2. 基于注解的声名事务控制

     <u>以下配置可用注解代替</u>

     ```xml
     <!--通知  事务的增强-->
         <tx:advice id="txAdvice" transaction-manager="transactionManager">
             <!--设置事务的属性信息-->
             <tx:attributes>
                 <!--配置切点的方法属性参数(配置事务参数:TransactionDefinition层)-->
                 <tx:method name="*" read-only="true" />
             </tx:attributes>
         </tx:advice>
         <!--aop织入-->
         <aop:config proxy-target-class="true">
             <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.service.Impl.selectServiceImpl.state(..))"/>
         </aop:config>
     ```

     1. **```@Transactional```**相当于```<tx:advice></tx:advice>```

        ```java
            @Transactional(transactionManager = "transactionManager",propagation = Propagation.REQUIRED,····)
        public void method(){·····}
        ```

        注：该注解可放于类名前和方法名前,类名和方法名前同时存在时，使用**就近原则**

        **```transactionManager```**属性可以不配

     2. ```spring.xml```添加事务注解驱动

        ```xml
            <!--事务注解驱动-->
        	<tx:annotation-driven proxy-target-class="true" transaction-manager="transactionManager"/>
        ```

### **<u>事务控制可以解决以下问题</u>**

```java
//当执行到int s = 1/0;报错,则不提交该方法的执行结果
	@Override
    public void test(String a, String b, Integer x, Integer y) {
        logInterface.a(a, b);
        int s = 1/0;
        logInterface.b(x, y);
    }
```



**<u>底层：编程式事务控制</u>**

