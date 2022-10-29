#### 一、ufw简介

  ufw是Ubuntu系列发行版自带的类似iptables的防火墙管理软件，底层也是基于netfilter的。ufw是Uncomplicated FireWall的简称，真正地简化了 iptables，它从出现的这几年，已经成为 Ubuntu 和 Debian 等系统上的默认防火墙。而且 ufw 出乎意料的简单，这对新管理员来说是一个福音，否则他们可能需要投入大量时间来学习防火墙管理。ufw默认关闭，开放所有端口。环境说明：如下操作命令和示例均在Ubuntu20.04系统中测试。

#### 二、使用示例

ufw命令需要通过root或者具有sudo权限的用户执行。

##### 1、获取ufw命令帮助

    root@s162:~# ufw --help

##### 2、版本查看

    root@s162:~# ufw version
    ufw 0.36
    Copyright 2008-2015 Canonical Ltd.

##### 3、防火墙服务启停管理

```bash
查看防火墙状态（运行/运行）
root@s162:~# ufw status
Status: active/inactive

To Action From
– ------ ----
80/tcp ALLOW Anywhere
80/tcp (v6) ALLOW Anywhere (v6)

88/tcp DENY OUT Anywhere
88/tcp (v6) DENY OUT Anywhere (v6)

查看防火墙状态详细信息

root@s162:~# ufw status verbose
Status: active
Logging: on (full)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To Action From
– ------ ----
80/tcp ALLOW IN Anywhere
80/tcp (v6) ALLOW IN Anywhere (v6)

88/tcp DENY OUT Anywhere
88/tcp (v6) DENY OUT Anywhere (v6)

启动防火墙并设置开机自启动

root@s162:~# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

停止防火墙并取消开机自启动

root@s162:~# ufw disable
Firewall stopped and disabled on system startup

重启防火墙

root@s162:~# ufw reload
Firewall reloaded

重置防火墙

root@s162:~# ufw reset
Resetting all rules to installed defaults. This may disrupt existing ssh
connections. Proceed with operation (y|n)? y
Backing up ‘user.rules’ to ‘/etc/ufw/user.rules.20210914_061746’
Backing up ‘before.rules’ to ‘/etc/ufw/before.rules.20210914_061746’
Backing up ‘after.rules’ to ‘/etc/ufw/after.rules.20210914_061746’
Backing up ‘user6.rules’ to ‘/etc/ufw/user6.rules.20210914_061746’
Backing up ‘before6.rules’ to ‘/etc/ufw/before6.rules.20210914_061746’
Backing up ‘after6.rules’ to ‘/etc/ufw/after6.rules.20210914_061746’
```

##### 4、防火墙日志管理

```bash
关闭防火墙日志

root@s162:~# ufw logging off
Logging disabled

开启防火墙日志

root@s162:~# ufw logging on
Logging enabled

设置防火墙日志级别为low

root@s162:~# ufw logging low
Logging enabled

设置防火墙日志级别为medium

root@s162:~# ufw logging medium
Logging enabled

设置防火墙日志级别为high

root@s162:~# ufw logging high
Logging enabled

设置防火墙日志级别为full

root@s162:~# ufw logging full
Logging enabled
```

##### 5、添加防火墙策略

如下示例以添加放行策略为例，示例同理适用allow、deny、reject，添加策略默认添加到入方向，如果需要添加到出方向使用关键词out。

```bash
添加放行一个端口策略

root@s162:~# ufw allow 8080
Rules updated
Rules updated (v6)

添加放行一个协议端口策略

root@s162:~# ufw allow 8081/tcp
Rules updated
Rules updated (v6)
root@s162:~# ufw allow 8082/udp
Rules updated
Rules updated (v6)

添加放行一个应用协议策略

root@s162:~# ufw allow http
Rule added
Rule added (v6)

添加放行一个端口策略

root@s162:~# ufw allow proto tcp to any port 81
Rule added
Rule added (v6)

允许指定源地址访问指定端口

root@s162:~# ufw allow proto tcp from 192.168.0.0/24 to 192.168.0.162 port 88
Rule added

运行指定源地址端口访问指定地址指定端口

root@s162:~# ufw allow proto udp from 1.2.3.5 port 5469 to 1.2.3.4 port 5469
Rule added

添加出方向安全策略

root@s162:~# ufw deny out 88/tcp
Rule added
Rule added (v6)
```

##### 6、删除防火墙策略

```bash
查看策略编号

root@s162:~# ufw status numbered
Status: active

To Action From
– ------ ----
[ 1] 80/tcp ALLOW IN Anywhere
[ 2] 80/tcp (v6) ALLOW IN Anywhere (v6)

删除指定编号策略

root@s162:~# ufw delete 2
Deleting:
allow 80/tcp
Proceed with operation (y|n)? y
Rule deleted (v6)

删除指定规则策略

root@s162:~# ufw delete allow 80/tcp
Rule deleted
Could not delete non-existent rule (v6)
```

##### 7、插入安全策略

```bash
root@s162:~# ufw insert 1 deny proto tcp from 192.168.0.1 to any port 80
Rule inserted
root@s162:~# ufw insert 2 deny from 192.168.0.1 to any port 80
Rule inserted
```

##### 8、添加默认安全策略

以下示例使用–dry-run参数模拟执行展示。

```bash
设置默认入策略为允许

root@s162:~# ufw --dry-run default allow
running ufw-init
running ufw-init
Default incoming policy changed to ‘allow’
(be sure to update your rules accordingly)

设置默认出策略为拒绝

root@s162:~# ufw --dry-run default deny outgoing
running ufw-init
running ufw-init
Default outgoing policy changed to ‘deny’
(be sure to update your rules accordingly)
```

##### 9、添加自定义应用

```bash
首先自定义一个应用

root@s162:/etc/ufw/applications.d# cat test
[test]
title=test
description=this is a test.
ports=81/tcp

添加策略时调用应用名称

root@s162:/etc/ufw/applications.d# ufw allow test
Rule added
Rule added (v6)

查看防火墙策略

root@s162:/etc/ufw/applications.d# ufw status
Status: active

To Action From
– ------ ----
80/tcp DENY 192.168.0.1
80 DENY 192.168.0.1
80/tcp REJECT Anywhere
test ALLOW Anywhere
80/tcp (v6) REJECT Anywhere (v6)
test (v6) ALLOW Anywhere (v6)

88/tcp DENY OUT Anywhere
88/tcp (v6) DENY OUT Anywhere (v6)
```

#### 三、ufw常用及默认规则说明

    ufw默认关闭
    ufw默认拒绝入流量，放行出流量
    ufw日志级别默认为low
    ufw添加策略默认为in放行，需要在出方向添加安全策略使用关键词out
    ufw添加策略为指定协议是默认为IP协议，包含TCP和UDP
    ufw使用–dry-run参数模拟执行

#### 四、ufw使用语法

```bash
ufw [–dry-run] enable|disable|reload

ufw [–dry-run] default allow|deny|reject [incoming|outgoing|routed]

ufw [–dry-run] logging on|off|LEVEL

ufw [–dry-run] reset

ufw [–dry-run] status [verbose|numbered]

ufw [–dry-run] show REPORT

ufw [–dry-run] [delete] [insert NUM] allow|deny|reject|limit [in|out] [log|log-all]
PORT[/PROTOCOL]

ufw [–dry-run] [rule] [delete] [insert NUM] allow|deny|reject|limit [in|out [on
INTERFACE]] [log|log-all] [proto PROTOCOL] [from ADDRESS [port PORT]] [to ADDRESS [port
PORT]]

ufw [–dry-run] route [delete] [insert NUM] allow|deny|reject|limit [in|out on INTERFACE]
[log|log-all] [proto PROTOCOL] [from ADDRESS [port PORT]] [to ADDRESS [port PORT]]

ufw [–dry-run] delete NUM

ufw [–dry-run] app list|info|default|update
```

#### 五、重要文件说明

```bash
# 参数配置文件/etc/ufw/ufw.conf
# 防火墙日志文件/var/log/ufw.log
# 路由转发参数配置文件/etc/ufw/sysctl.conf
# 应用定义文件目录/etc/ufw/applications.d
```