#### 1.JDK的安装配置

下载[jdk8-linux-x64](https://download.oracle.com/otn/java/jdk/8u331-b09/165374ff4ea84ef0bbd821706e29b123/jdk-8u331-linux-x64.tar.gz?AuthParam=1650616285_8356da3e1a80e222e450ba843107c4c8)

使用SFTP上传文件到`/usr/java/jdk`,建议解压缩包都在opt里(权限访问问题不严重)

```bash
#进入压缩包所在目录
cd /opt/java
#解压压缩包
sudo tar zxvf jdk-8u331-linux-x64.tar.gz
#得到一个新的文件夹jdk1.8.0_331
#修改文件夹名为jdk8
sudo mv jdk1.8.0_331 jdk8
#删除压缩包文件
sudo rm -f jdk-8u331-linux-x64.tar.gz
#切换到home目录下
cd
#配置环境变量,找到.bashrc文件(一定是home目录下)
#显示下的所有文件
ll -a
###配置当前用户的jdk环境
#进入配置文件
sudo vim ~/.bashrc
#在最后一行添加jdk的环境变量
export JAVA_HOME=/usr/java/jdk8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
#保存退出
:wq
#重新加载环境变量
source ~/.bashrc
#测试
java -version

###配置全局变量
#进入配置文件
sudo vim /etc/profile
#在最后一行也添加jdk的环境变量
export JAVA_HOME=/usr/java/jdk8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
#保存退出
:wq
#重启机器或执行命令，是环境生效
source /etc/profile
#测试
java -version

###把jdk作为服务直接运行
#--install /usr/bin/java java -->创建一个(类似快捷键)的java
#/opt/java/jdk8/bin/ava  -->使前面中创建的这个文件映射过来(JAVA_HOME/bin里的java运行程序)
sudo update-alternatives --install /usr/bin/java java /usr/java/jdk8/bin/java 300
sudo update-alternatives --install /usr/bin/javac javac /usr/java/jdk8/bin/javac 300
```

#### 2.Tomcat的安装配置

下载[tomcat9](https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.62/bin/apache-tomcat-9.0.62.tar.gz)(Core下的.tar.gz)

创建组和管理员

```bash
##创建管理tomcat组下的管理员tomcatadmin
#创建组
sudo groupadd tomcat
#创建tomcatadmin
#useradd -s /bin/false -->不能shell登录(效果和/usr/sbin/nologin一样)
#-g tomcat -->设置该用户属于哪个group
#-d /usr/apache/tomcat -->设置该用户的home目录
#tomcatadmin -->创建的用户名
sudo useradd -s /bin/false -g tomcat -d /usr/apache/tomcat tomcatadmin
#设置tomcatadmin的管理员密码
sudo passwd tomcatadmin
#New password:
#Retype new password:
#passwd: password updated successfully
```

上传Tomcat文件

```bash
##用SFTP上传tomcat的压缩包到/usr/apache/tomcat里
#解压(不建议解压在usr/lib里,权限问题)
cd /usr/apache/tomcat
sudo tar zxvf apache-tomcat-9.0.62.tar.gz
#删除压缩文件
sudo rm -f apache-tomcat-9.0.62.tar.gz
#把文件夹(TOMCAT_HOME)改名
sudo mv apache-tomcat-9.0.62 tomcat9

###权限
#设置TOMCAT_HOME及以内的所有文件属于tomcat组
sudo chgrp -R tomcat /usr/apache/tomcat/tomcat9
#进入tomcat9
cd tomcat9
#conf下的文件(包括conf):所属的组可以读取
sudo chmod -R g+r conf
#conf在这个文件夹:所属组可以执行
sudo chmod g+x conf
#webapps,work,temp,logs及其以下的文件拥有者为tomcatadmin
sudo chown -R tomcatadmin webapps/ work/ temp/ logs/
#权限不足的问题，百度
```

修改`TOMCAT_HOME/bin`里的`startup.sh`和`shutdown.sh`

```bash
#root进入
cd bin
vi startup.sh
vi shutdown.sh
#在最后一行前添加以下信息
export JAVA_HOME=/usr/java/jdk8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

export TOMCAT=/usr/apache/tomcat/tomcat9

#这是最后一行
exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```



创建Tomcat服务文件`sudo vim /etc/systemd/system/tomcat.service`

`tomcat.service`文件内容如下:

```bash
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/jdk/jdk8
Environment=CATALINA_PID=/usr/apache/tomcat/tomcat9/temp/tomcat.pid
Environment=CATALINA_HOME=/usr/apache/tomcat/tomcat9
Environment=CATALINA_BASE=/usr/apache/tomcat/tomcat9
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/usr/apache/tomcat/tomcat9/bin/startup.sh
ExecStop=/usr/apache/tomcat/tomcat9/bin/shutdown.sh

User=tomcatadmin
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

完成后，保存并关闭文件。重新加载systemd守护程序，以便它可以找到我们的新服务文件：

```bash
sudo systemctl daemon-reload
```

启动tomcat服务

```bash
sudo systemctl start tomcat
//关闭tomcat
sudo systemctl stop tomcat
```

查看tomcat运行状态

```bash
sudo systemctl status tomcat
```

开放端口`sudo ufw allow 8080`浏览器测试:

```java
http://ip:8080
```

把tomcat添加到服务器启动时就启动

```bash
sudo systemctl enable tomcat
```

**Tomcat**有一个Web管理器应用程序。为了使用它，我们需要在**tomcat-users.xml**文件中设置身份验证。

```xml
#TOMCAT_HOME=/usr/apache/tomcat/tomcat9
#该配置文件在TOMCAT_HOME里的conf下,99%都是注释,直接编辑
#在约束文件里添加
    <!--添加一个能够访问管理器和管理界面的用户-->
	<user username="username" password="password" roles="manager-gui,admin-gui"/>

```

··········



3.mysql

```bash
#访问源列表(资源镜像:阿里,腾讯,华为，····)里的每个网址，并读取软件列表，然后保存在本地电脑。
sudo apt update
#把本地已安装的软件，与刚下载的软件列表里对应软件进行对比，如果发现已安装的软件版本太低，就会提示你更新,一般都会加-y表示直接更新
sudo apt upgrade -y
#清理无用的包sudo apt-get clean && sudo apt-get autoclean
sudo apt-get clean
#安装mysql
sudo apt install mysql-server
-y确认
#安装完成后mysql会自动运行
#查看状态
sudo systemctl status mysql
····(傻瓜式安装)
#为了提高MySQL安装的安全性，执行:
sudo mysql_secure_installation
#选择root密码强度等级(LOW=0,MEDIUM=1,STRANG=2)
#禁止远程系统的root登录
#删除test数据库
#重新加载权限表
```















