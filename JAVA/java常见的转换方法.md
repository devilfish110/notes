### 1.把String转为java.sql.Date

###### 利用父类转换

1.首先用java.text.SimpleDateFormat对象设置时间格式

```java
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
```

2.SimpleDateFormat中有一个方法:`parse(String text,  ParsePosition pos);` ，能够从字符串中解析文本以产生一个 `java.util.Date` 。

```
java.util.Date date1 = dateFormat.parse("2022-4-6");//这里为String
```

3.得到date1,我们就可以使用`long  time = date1.getTime();

返回以1970年1月1日为准的00:00:00 GMT的毫秒数。(反正很长)

```
long time = date1.getTime();//1649174400000
```

4.这样我们便可以new一个java.sql.Date的对象date2把上面得到的毫秒数传进去,date2自动计算生成当前时间(yyyy-MM-dd)

```
java.sql.Date date2 = new Date(time);//2022-4-6
```

到此成功把字符串转为java.sql.Date类型，解决问题！

