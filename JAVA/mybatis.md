##### ```mappers.xml```配置文件

- ```<select></select>```

- ```<insert></insert>```

  ```bash
  //开启插入语句后回写自动递增的id字段数据，dao层方法返回值为void
  useGeneratedKeys="true" keyProperty="id"
  ```

  执行插入后输出结果：

  ```java
          registerUserDao registerUserDao = sqlSession.getMapper(registerUserDao.class);
          User user = new User(null, "test", "test");
          registerUserDao.register(user);
          System.out.println(user);//User{id=7, loginName='test', password='test'}
  ```

- ```<update></update>```

- ```<delete></delete>```

### **```TypeHandler```**数据类型转换器

自定义转换类继承```BaseTypeHandler```接口实现方法

```java
//把java的数据类型转为数据库的类型setNonNullParameter
//查询后把数据库的类型转为java的数据类型getNullableResult
```

1. ```java
   package com.typeConverter;
   
   import org.apache.ibatis.type.BaseTypeHandler;
   import org.apache.ibatis.type.JdbcType;
   
   import java.sql.CallableStatement;
   import java.sql.PreparedStatement;
   import java.sql.ResultSet;
   import java.sql.SQLException;
   import java.util.Date;
   
   /**
    * java.date和Long的转换
    */
   public class dateConverter extends BaseTypeHandler<Date> {
       /**参数解释
       *PreparedStatement ps/ResultSet rs/CallableStatement cs:结果集
       *int i:需要赋值的索引位置
       *Date parameter:需要转换的类型
       *JdbcType jdbcType:jdbc的数据类型
       *String columnName:字段名称
       *int columnIndex:字段索引
       *
       */
       //结果集转为java数据类型
       @Override
       public void setNonNullParameter(PreparedStatement ps, int i, Date parameter, JdbcType jdbcType) throws SQLException {
           //获取时间毫秒值
           long time = parameter.getTime();
           //结果集转换
           ps.setLong(i,time);
       }
       //根据字段名获取
       @Override
       public Date getNullableResult(ResultSet rs, String columnName) throws SQLException {
           //获取需要的字段数据
           long aLong = rs.getLong(columnName);
           //把结果集转为date
           Date date = new Date(aLong);
           return date;
       }
       
       //根据索引位置获取
       @Override
       public Date getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
           return null;
       }
       
       //根据存储过程获取
       @Override
       public Date getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
           long aLong = cs.getLong(columnIndex);
           Date date = new Date(aLong);
           return date;
       }
   }
   ```

2. ```mybatis-config.xml```配置添加自定义类型处理器

   ```xml
       <!--自定义类型处理器-->
       <typeHandlers>
           <typeHandler handler="com.typeConverter.dateConverter"/>
           <!--以包为单位导入-->
           <package name="com.typeConverter"/>
       </typeHandlers>
   ```


##### 注解开发



- ```@Insert```

  配合```@Options```使用
