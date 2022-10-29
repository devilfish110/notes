#### 目录：

- DDL：操作数据库，表
- DQL:查询表中的数据
- DML：增删改表中的数据
- DCL：数据库用户权限



##### 一、DQL:()

###### **```select 字段列表 from 表名列表 where条件 group by 分组字段 having 分组后的条件 order by 排序条件 limit 分页限定;```**

1. 排序查询：```ORDER BY```
2. 聚合函数:```GROUP BY```
3. 分组查询
4. 分页查询

- 排序查询

  ```mysql
  #select * from 表名 ORDER BY 字段1,字段2 排序方式;
  select * from student ORDER BY math ASC;#按照math升序排列
  select * from student ORDER BY math DESC;#按照math降序排列
  #按照math和english升序排列,当math一样时才使用第二排序
  select * from student ORDER BY math,english ASC;
  ```

- 聚合函数

  **概念**:将一列数据最为一个整体进行纵向计算。(如平均分)

  **注意:计算时排除Null值**

  1. count:计算个数，一般为非空字段(主键)

  2. max:最大值

  3. min:最小值

  4. sum:和

  5. avg:平均值

     ```mysql
     #####select 函数名(字段名) from 表名;
     #select COUNT(字段名) from 表名;
     select count(math) from student;
     #如果有NULL值则替换为0
     select count(IFNULL(math,0)) from student;
     #最大值/最小值/求和/平均分
     #select max/min/sum/avg(字段名) from 表名;
     select max(math) from student;
     select min(math) from student;
     select sum(math) from student;
     select avg(math) from student;
     ```

- 分组查询

  ```mysql
  #####select 字段名 from 表名 where条件 GROUP BY 分组字段 HAVING 聚合函数 条件;
  #####select 字段名1,聚合函数名(字段名2) from 表名 GROUP BY 字段名1;
  #按照性别分组
  select gender from xsb GROUP BY gender;
  #按照gender分组并计算math的平均分
  select gender,AVG(math) from xsb GROUP BY gender;
  ```

  **where和HANIN条件判断区别:**

  1. where在分组前限定，HAVING在分组后
  2. where后不能跟聚合函数,HAVING后可以跟聚合函数

- 分页查询

  ```mysql
  #####LIMT 开始索引,每页的条数;
  select * from student LIMIT 0,2;#从第一条数据开始,每页显示2条数据
  select * from student LIMIT 2,2;#第二页
  #开始索引=(当前页码-1) * 每页显示的条数
  ```

除去重复结果:distinct

**```select distinct 字段列表 from 表名;```**

判断字段是否为null：IFNULL函数

```mysql
#####IFNULL(可能为NULL的字段,替换值)
select name,math,english,math + ifnull(english,0) as 总成绩 from student;
```



##### 二、约束

- 主键约束 PRIMARY KEY：值不能为NULL且不能重复
- 非空约束 NOT NULL：值不能为NULL
- 唯一约束 UNIQUE：值不能重复,但可以为NULL
- 外键约束 FOREIGN KEY：另一张表的主键作为当前表的外键
- 自动增长约束 AUTO_INCREMENT:值自动递增

```mysql
#修改约束
ALTER TABLE 表名 MODIFY 字段名 字段类型 约束;
#删除约束
ALTER TABLE 表名 MODIFY 字段名 字段类型;
#删除唯一约束
ALTER TABLE 表名 DROP INDEX 字段名;
#删除主键约束
ALTER TABLE 表名 DROP PRIMARY KEY;
#外键约束
CONSTRAINT 外键名称 FOREIGN KEY (外键字段名) REFERENCES 外键所在表名(字段名)
##创建表时添加外键约束
create table dept(
    dept_id int PRIMARY KEY AUTO_INCREMENT,#
)
create table employee(
	emp_id int PRIMARY KEY AUTO_INCREMENT,#主键字段
    dept_id int,#外键字段
    CONSTRAINT emp_dept_fk FOREIGN KEY (dept_id) REFERENCES dept(dept_id)
)
##添加外键约束
ALTER TABLE 表名 ADD FOREIGN KEY CONSTRAINT 外键名称 FOREIGN KEY (外键字段名) REFERENCES 外键所在表名(字段名);
#删除外键约束
ALTER TABLE 表名 DROP FOREIGN KEY 外键名称;
```

##### 三、**DCL**

```mysql
-- 切换mysql数据库
use mysql;
select * from user;-- 查询数据库用户表
-- Host 允许登录的主机ip(%表示所有)
-- 添加数据库用户
create user '用户名'@'主机名' IDENTIFIED BY '密码';
-- 删除用户
drop user '用户名'@'主机名';
-- 修改用户密码,password()函数是mysql自带的密码加密函数,5.7.9以后废除
update user set password = password('新密码') where user ='用户名';
-- 简化
set password for '用户名'@'主机名' = password('新密码');
-- mysql8
alter user '用户名'@'主机名' IDENTIFIED BY '新密码';

-- 授权 --

-- 查询用户权限
use mysql;
show GRANTS for '用户名'@'主机名';
use mysql;
select User,authentication_string,Host from user;
-- 授权给用户
GRANT 权限列表 on 数据库名 to '用户名'@'主机名'; # 数据库授予用户
GRANT 权限列表 on 数据库名.表名 to '用户名'@'主机名'; # 表授予用户
GRANT REPLICATION SLAVE on *.* to '用户名'@'主机名'; # 授予用户SLAVE权限(主从复制使用)

-- 修改用户远程访问权限
# *.*代表所有资源权限(select/delete/insert/update···)用户名:user  密码:password  '%'代表所有主机(也可以为具体的ip) WITH GRANT OPTION:允许级联授权
GRANT ALL PRIVILEGES ON *.* TO '用户名'@'%' IDENTIFIED BY '密码' WITH GRANT OPTION;
#刷新数据库
FLUSH PRIVILEGES;
#查看权限
select User,authentication_string,Host from user;

-- 授权所有权限,所有库，表
GRANT ALL on *.* to '用户名'@'主机名';

-- 撤销权限
revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';
```

###### **root用户密码忘了**

```bash
#1.cmd停止mysql服务
net stop mysql
#2.无验证方式启动mysql服务
mysql --skip-grant-tables
#再打开一个cmd窗口输入mysql,然后改root密码
```



##### 四、MySQL主从复制操作

1. ###### 主库中创建用户并授予```REPLICATION SLAVE```权限(用于从库复制数据)

   ```mysql
   -- 添加数据库用户
   create user '用户名'@'主机名' IDENTIFIED BY '密码';
   -- 授予用户SLAVE权限(主从复制使用)
   GRANT REPLICATION SLAVE on *.* to '用户名'@'主机名';
   ```

2. ###### 设置数据库(主库)的配置文件开启二进制日志，设置serverID(唯一标识，不能与从库一样)

   windows下为```my.ini```文件，Linux下为```/etc/my.conf```文件

   ```tex
   # 开启log-bin二进制日志,设置日志文件位置
   log_bin=E:\SQL_Data\mysql_log
   # 配置唯一的服务器ID
   server-id=1
   ```

   ```bash
   # 重启mysql
   systemctl restart mysqld
   ```

   ```mysql
   # 查看主库设置的值,看是否设置成功
   show VARIABLES LIKE 'server_id';
   show VARIABLES LIKE 'log_bin';
   ```

3. ###### 查看主库状态，记录File(日志文件名)和position(记录的位置,数字，主库执行操作时会发生改表)

   ```mysql
   # 查看主库状态
   show master status;
   ```

4. ###### 修改从库的mysql配置

   ```tex
   [mysqId]
   server-id=101 # 不能和主库相同
   ```

5. ###### 重启从库的mysql

6. ###### 设置从库怎么连接到主库复制数据

   ```mysql
   # 从库中设置用于复制数据库的用户及二进制文件
   CHANGE master to MASTER_host='主机地址（ip）', MASTER_user='主库创建的用户名',master_password='密码',MASTER_log_file='主库查到的File',MASTER_log_pos=主库查到的position;
   # 开启从库
   start slave;
   ```

7. ###### 查看从库的状态

   ```mysql
   # 查看从库状态
   show slave status;
   -- Slave_IO_state:Waiting for master to send event
   -- Slave_IO_runing:yes
   -- Slave_SQL_runing:yes
   ### 如果以上数据不是yes则出异常了
   ```

   异常：
   
   ```mysql
   ### Slave_SQL_Running: No
   -- 1.程序可能在slave上进行了写操作
   -- 2.也可能是slave机器重起后，事务回滚造成的.
   -- 一般是事务回滚造成的：
   -- 解决办法：
   stop slave;
   set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
   start slave;
   ```
   
   



