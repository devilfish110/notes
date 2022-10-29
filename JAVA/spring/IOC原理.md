### 三层(`dao`/`service`/`ontroller`)的bean注入空指针异常问题

#### 一、简单的注入：

##### 1.**`set`**方法注入

**`dao`**层的代码:

```java
//dao接口
package com.dao;

public interface selectDao {
    public void select();
}
//dao实现类
package com.dao;
public class daoImpl implements selectDao{
    @Override
    public void select() {
        System.out.println("dao实现类");
    }
}
```

**`service`**层代码

```java
//接口
package com.service;
public interface select {
    public void select();
}
//实现类
package com.service;

import com.dao.selectDao;

public class serviceImpl implements select{
    //注入接口
    private selectDao dao;
    public void setDao(selectDao dao) {
        this.dao = dao;
    }
    @Override
    public void select() {
        dao.select();
        System.out.println("service实现类");
    }
}
```

`application-context.xml`配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!--注入bean-->
    <bean class="com.dao.daoImpl" id="selectDao"/>
    <bean class="com.service.serviceImpl" id="service">
        <!---->
        <property name="dao" ref="selectDao"/>
    </bean>
</beans>
```

##### 2.**`有参构造`**方法注入

**`dao`**层不变

**`service`**层实现类创建有参构造，参数为**`dao`**层的接口

```java
package com.service;

import com.dao.selectDao;

public class serviceImpl implements select{
    //有参构造注入
    private selectDao dao;
    public serviceImpl(selectDao dao) {
        this.dao = dao;
    }
    @Override
    public void select() {
        dao.select();
        System.out.println("service实现类");
    }
}
```

**`application-context.xml`**修改告知spring的方法

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.dao.daoImpl" id="selectDao"/>
    <bean class="com.service.serviceImpl" id="service">
        <!--有参构造参数告知spring-->
        <constructor-arg name="dao" ref="selectDao"/>
    </bean>
</beans>
```

#### 二、注解注入

**`dao`**层实现类(类上添加注解`@Repository`)

```java
package com.dao;

import org.springframework.stereotype.Repository;

//dao实现类
@Repository(value = "selectDao")
public class daoImpl implements selectDao{
    @Override
    public void select() {
        System.out.println("dao实现类");
    }
}
```

**`service`**层实现类

```java
package com.service;

import com.dao.selectDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service("service")
public class serviceImpl implements select{

    //有参构造注入
    //或者@Resource
    @Autowired
    @Qualifier(value = "selectDao")
    private selectDao dao;
    
    @Override
    public void select() {
        dao.select();
        System.out.println("service实现类");
    }
}
```

**`application-context.xml`**添加`context`命名空间，实现注解扫描:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--扫描注解-->
    <context:component-scan base-package="com"/>
</beans>
```

#### **<u>获取```bean```使用</u>**

**只能```new ClassPathXmlApplicationContext```或者其他三个方法,不能通过注入使用**

```java
	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-context.xml");
	serviceImpl service = context.getBean("service",serviceImpl.class);
	service.select();
```
