```tex
#FTP的运行模式
#YES：FTP服务器以standalone模式运行,效率快，耗资源
#NO：FTP服务器以inetd(xinetd)模式运行,一般不耗资源(默认值)
listen=NO

#FTP服务监听的port
listen_port=21

#监听ipv6的sockets
listen_ipv6=YES


# 是否允许匿名登陆FTP(默认：NO).
anonymous_enable=NO


#开启本地用户登录
local_enable=YES


#开启全局写入权限
write_enable=YES


#本地用户新增档案时的umask值(默认：077)多数使用022
#local_umask=022

#本地用户上传文件后文件的(umask)权限
file_open_mode=0755


# 开启全局写入权限后，匿名用户是否可以上传文件
#anon_upload_enable=YES


# 开启全局写入权限后，允许匿名用户新增目录
#anon_mkdir_write_enable=YES


# 使用者第一次进入一个目录时，FTP检查是否有.message文件，
#有则出现文件里的内容(通常为欢迎语句或是说明)
dirmessage_enable=YES


#显示带有本地时区时间的目录列表。 默认设置为显示 GMT。
# MDTM FTP 命令返回的时间也受此选项影响。
use_localtime=YES

#确保FTP服务使用20端口传输数据
connect_from_port_20=YES

#FTP服务器数据传输端口为20
ftp_data_port=20


#设置是否改变匿名用户上传文件(非目录)的属主。
#chown_uploads=YES

#设置匿名用户上传文件的属主名(用户)。建议不能为root
#chown_username=whoever


# 600秒不对FTP服务器操作则断开连接
idle_session_timeout=600


#建立FTP数据连接的超市时间为120秒
data_connection_timeout=120

#最大的用户连接数量(0表示无上限)
max_clients=5

#控制每个ip可以与FTP建立几个连接
max_per_ip=1

#本地用户使用的最大传输速度,单位：b/s (0表示不限制)
local_max_rate=0

#指定一个非特权用户，用于运行 vsftpd
#nopriv_user=ftpsecure


# 服务器是否识别异步ABOR请求(不安全).the code is bad.
#不启用他旧的FTP客户端无法正常连接
#async_abor_enable=YES


#启用ASCII模式传输数据 
ascii_upload_enable=YES
ascii_download_enable=YES

#欢迎话语(String)
ftpd_banner=Welcome to taotao FTP service


# 防(DOS)开启匿名登录时使用email文件里的地址登录,不在地址里不能登录
#deny_email_enable=YES
# email文件位置(默认/etc/vsftpd.banned_email)
#banned_email_file=/etc/vsftpd.banned_email

#chroot_list里的本地用户是否可以切换到上级目录
chroot_local_user=YES
#启用加载chroot_list文件
chroot_list_enable=YES
# (chroot_list默认位置/etc/vsftpd.chroot_list)
chroot_list_file=/etc/vsftpd.chroot_list


# 是否允许登录者使用ls -R(查看当前目录下，子目录中的文件)
#ls_recurse_enable=YES

# This option should be the name of a directory which is empty.  Also, the
# directory should not be writable by the ftp user. This directory is used
# as a secure chroot() jail at times vsftpd does not require filesystem
# access.
secure_chroot_dir=/var/run/vsftpd/empty

# vsftpd使用的 PAM 服务名称
pam_service_name=vsftpd

# SSL证书
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO


#vsftpd 是否使用 utf8 文件体系.
utf8_filesystem=YES


#启用传输日志记录
xferlog_enable=YES

# 将日志文件写成xferlog的标准形式
#xferlog_std_format=YES

# 日志保存的位置
#xferlog_file=/var/log/vsftpd.log

#配置ftp服务器的文件传输保存位置
local_root=/home/ftp/ftpfile

```

