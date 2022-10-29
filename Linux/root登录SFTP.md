修改root能够登录SFTP

下载`openssh-server`

修改ssh的配置文件

```bash
#输入命令sudo vi /etc/ssh/sshd_config打开配置文件
username@ip:~$ cat /etc/ssh/sshd_config
#找到PermitRootLogin prohibit-password修改为
PermitRootLogin yes
#保存
:wq
```

重启SSH服务

```bash
sudo service sshd reload
```

测试root登录SFTP





