---
title: 安装OpenLDAP+phpLDAPadmin
toc: true
date: 2019-09-13 01:14:39
tags:
categories:
---



## 一、基础设置

### 1.1 环境说明

```
Centos 7.5
openldap 2.4.44
```

### 1.2 关闭防火墙和selinux

```
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld.service && systemctl disable firewalld.service
firewall-cmd --state
```

### 1.3 更新yum源

```
wget http://mirrors.aliyun.com/repo/Centos-7.repo -O /etc/yum.repos.d/Centos-7.repo
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
mv /etc/yum.repos.d/Centos-7.repo /etc/yum.repos.d/CentOS-Base.repo
yum clean all
yum makecache
```

 

## 二、安装 OpenLDAP

### 2.1 安装openldap

```
yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel migrationtools
```

查看版本：slapd -VV

### 2.2 生成管理员密码

```
slappasswd -s Admin123

{SSHA}qtkKhiajMDZpbAS9sS9K4TfnePglsVz4
```

管理员密码为：Admin123，下面是对密码进行加密后的字符串。

### 2.3 修改olcDatabase={2}hdb.ldif文件

从OpenLDAP2.4.23版本开始所有配置数据都保存在/etc/openldap/slapd.d/中，建议不再使用slapd.conf作为配置文件

```
vim /etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif

#修改
olcSuffix: dc=wmqe,dc=com
olcRootDN: cn=admin,dc=wmqe,dc=com
#添加
olcRootPW: {SSHA}qtkKhiajMDZpbAS9sS9K4TfnePglsVz4
```

注意：其中cn=admin中的admin表示OpenLDAP管理员的用户名，而olcRootPW表示OpenLDAP管理员的密码。

### 2.4 修改olcDatabase={1}monitor.ldif文件

```
vim /etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif

#修改管理员信息
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=extern
al,cn=auth" read by dn.base="cn=admin,dc=wmqe,dc=com" read by * none
```

### 2.5 验证配置

slaptest -u

```
5d24c09b ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif"
5d24c09b ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif"
config file testing succeeded
```

### 2.6 启动 OpenLDAP

```
systemctl start slapd
systemctl enable slapd
systemctl status slapd
```

启动后监听 389 端口

 

## 三、配置 OpenLDAP

### 3.1 配置OpenLDAP数据库

OpenLDAP默认使用的数据库是BerkeleyDB，现在来开始配置OpenLDAP数据库，使用如下命令：

```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap:ldap -R /var/lib/ldap
chmod 700 -R /var/lib/ldap
ll /var/lib/ldap/
clip_image010
```

注意：/var/lib/ldap/就是BerkeleyDB数据库默认存储的路径。

### 3.2 导入基本Schema

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

### 3.3 修改migrate_common.ph文件

migrate_common.ph文件主要是用于生成ldif文件使用。

```
vim /usr/share/migrationtools/migrate_common.ph

$DEFAULT_MAIL_DOMAIN = "wmqe.com";
$DEFAULT_BASE = "dc=wmqe,dc=com";
$EXTENDED_SCHEMA = 1;
#重启
systemctl restart slapd
```

到此OpenLDAP的配置就已经全部完毕。

 

## 四、添加用户和组

默认情况下OpenLDAP是没有普通用户的，只有一个管理员用户；管理用户就是前面配置的 cn=admin,dc=wmqe,dc=com 。

### 4.1 创建用户和组

现在我们把系统中的用户，添加到OpenLDAP中。为了进行区分，我们现在新加两个用户ldapuser1和ldapuser2，和两个用户组ldapgroup1和ldapgroup2，如下：

```
groupadd ldapgroup1
groupadd ldapgroup2
useradd -g ldapgroup1 ldapuser1
useradd -g ldapgroup2 ldapuser2
passwd ldapuser1
passwd ldapuser2
```

### 4.2 写入到文件

把刚刚添加的用户和用户组属性信息提取出来

```
grep "ldapuser" /etc/passwd > /root/users
grep "ldapgroup" /etc/group > /root/groups
```

### 4.3 生成ldif文件

上述生成的用户和用户组属性，使用migrate_passwd.pl文件生成要添加用户和用户组的ldif

```
/usr/share/migrationtools/migrate_passwd.pl /root/users > /root/users.ldif
/usr/share/migrationtools/migrate_group.pl /root/groups > /root/groups.ldif
cat users.ldif
cat groups.ldif
```

注意：后续如果要新加用户到OpenLDAP中的话，我们可以直接修改users.ldif文件即可，或者采用后续需要安装的phpLDAPadmin工具添加。

### 4.4 新建基础数据库ldif文件

vim /root/base.ldif

```
dn: dc=wmqe,dc=com
o: wmqe com
dc: wmqe
objectClass: top
objectClass: dcObject
objectclass: organization

dn: cn=admin,dc=wmqe,dc=com
cn: admin
objectClass: organizationalRole
description: Directory Manager

dn: ou=People,dc=wmqe,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=wmqe,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit
```

注意格式：ldif 文件以空行作为用户分割，格式要保持一致。

### 4.5 导入账号信息到OpenLDAP数据库

1）导入基础数据库

```
ldapadd -x -w Admin123 -D cn=admin,dc=wmqe,dc=com -f /root/base.ldif
```

2）导入用户信息

```
ldapadd -x -w Admin123 -D cn=admin,dc=wmqe,dc=com -f /root/users.ldif
```

3）导入用户组信息

```
ldapadd -x -w Admin123 -D cn=admin,dc=wmqe,dc=com -f /root/groups.ldif
```

同时查看BerkeleyDB数据库文件中多了cn.bdb、sn.bdb、ou.bdb等数据库文件

```
ll /var/lib/ldap/
```

### 4.6 用户加入到用户组

目前OpenLDAP用户和用户组之间是没有任何关联的，需要新建添加用户到用户组的ldif文件。

示例：把ldapuser1用户加入到ldapgroup1用户组。

1）新建文件

```
cat > add_user_to_groups.ldif << EOF
dn: cn=ldapgroup1,ou=Group,dc=wmqe,dc=com
changetype: modify
add: memberuid
memberuid: ldapuser1
EOF
```

2）添加

```
ldapadd -x -w Admin123 -D cn=admin,dc=wmqe,dc=com -f /root/add_user_to_groups.ldif
```

3）查看

```
ldapsearch -LLL -x -w Admin123 -D 'cn=admin,dc=wmqe,dc=com' -b 'dc=wmqe,dc=com' cn='ldapgroup1'
#下面输出信息可看到ldapgroup1组包含用户为ldapuser1

dn: cn=ldapgroup1,ou=Group,dc=wmqe,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup1
userPassword:: e2NyeXB0fXg=
gidNumber: 1000
memberUid: ldapuser1
```

到这里，基本功能已经配置完成，可以通过phpLDAPadmin进程访问连接。

 

## 五、其他功能配置

### 5.1 开启日志功能

默认情况下OpenLDAP是没有启用日志记录功能的，但是在实际使用过程中，我们为了定位问题需要使用到OpenLDAP日志。
1）新建日志配置ldif文件：

```
cat > /root/loglevel.ldif << EOF
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: stats
EOF
```

2）导入到OpenLDAP中

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/loglevel.ldif
```

3）重启OpenLDAP服务

```
systemctl restart slapd
```

4）修改rsyslog配置文件

```
cat >> /etc/rsyslog.conf << EOF
local4.* /var/log/slapd.log
EOF
```

5）并重启rsyslog服务

```
systemctl restart rsyslog
```

6）查看OpenLDAP日志

```
tail -f /var/log/slapd.log
```

现在查看会提示文件不存在，需要对ldap进行操作后可以看到有日志输出。

### 5.2 配置SSL

通过网络访问 OpenLDAP 服务器，明文传输这些数据存在被他人嗅探的风险。本节设置 LDAP 服务器与客户端之间的 SSL 连接以加密传输数据。
参考：https://docs.oracle.com/cd/E52668_01/E54669/html/ol7-s9-auth.html

1）停止服务

```
systemctl stop slapd
```

2）开启SSL

```
vim /etc/sysconfig/slapd

SLAPD_LDAPS=yes
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
```

3）启动服务

```
systemctl start slapd
```

4）查看已监听 636 端口

```
netstat -tulnp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:636             0.0.0.0:*               LISTEN      2132/slapd          
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      2132/slapd          
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      819/sshd            
tcp6       0      0 :::636                  :::*                    LISTEN      2132/slapd          
tcp6       0      0 :::389                  :::*                    LISTEN      2132/slapd          
tcp6       0      0 :::80                   :::*                    LISTEN      818/httpd           
tcp6       0      0 :::22                   :::*                    LISTEN      819/sshd 
```

后续对接可以采用加密方式连接：ldaps://192.168.159.130:636。

 

启动服务报错解决：

```
报错：unable to open file “/var/run/openldap/slapd.args”: 13 (Permission denied)
解决：创建/var/run/openldap/slapd.args并赋予777权限

报错：unable to open file “/var/run/openldap/slapd.pid”: 13 (Permission denied)
解决：创建/var/run/openldap/slapd.pid并赋予777权限
```

因为异常结束了服务进程，导致有文件残留，需要手动创建并赋予777权限，后续正常关闭服务这两个文件都会自动被删除。

### 5.3 禁用匿名访问

参考：https://www.ilanni.com/?p=14035
默认openldap在匿名情况下是可以被访问的，而且openldap的相关信息，除了用户的密码信息之外，其他openldap的信息完全被呈现出来。
1）新建文件

```
cat > /root/disable_anon.ldif << EOF
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon

dn: cn=config
changetype: modify
add: olcRequires
olcRequires: authc

dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcRequires
olcRequires: authc
EOF
```

2）导入文件

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /root/disable_anon.ldif
```

不用重启服务即可生效

 

## 六、安装 phpLDAPadmin

### 6.1 安装Apache PHP

```
yum -y install httpd php php-ldap php-gd php-mbstring php-pear php-bcmath php-xml
```

### 6.2 安装 phpldapadmin

```
yum --enablerepo=epel -y install phpldapadmin
```

### 6.3 修改phpldapadmin配置文件

```
vim /etc/phpldapadmin/config.php

#打开 dn 注释，注释掉uid
$servers->setValue('login','attr','dn');
// $servers->setValue('login','attr','uid');
```

phpldapadmin默认使用的是uid方式进行登录，改为dn认证。

### 6.4 修改httpd配置文件

修改httpd与phpldapadmin集成的配置文件，把httpd与phpldapadmin进行集成。

```
vim /etc/httpd/conf.d/phpldapadmin.conf

Require all granted
```

将Require local 改为 Require all granted

### 6.5 启动httpd

```
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

监听80端口

访问：http://192.168.159.130/phpldapadmin，登入账号：cn=admin,dc=wmqe,dc=com， 密码：Admin123

登入后可以看到已经有之前创建的用户和组了

##  七、自助修改密码系统

### 7.1 安装Self Service Password

1）配置Self Service Password的yum仓库源

```
cat >> /etc/yum.repos.d/ltb-project.repo << EOF
[ltb-project-noarch]
name=LTB project packages (noarch)
baseurl=https://ltb-project.org/rpm/\$releasever/noarch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-LTB-project
EOF
```

2）安装

```
yum -y install self-service-password
```

查看下Self Service Password安装的文件，如下：

```
rpm -ql self-service-password
```

看出被安装到 /usr/share/self-service-password 目录下，其中 config.inc.php 是 Self Service Password 的配置文件。

### 7.2 修改配置文件

1）修改apache配置文件

前面phpLDAPadmin采用apache，这里就不用再次安装了，直接修改配置文件就行。（也可以采用nginx）

```
cp /etc/httpd/conf.d/self-service-password.conf /etc/httpd/conf.d/self-service-password.conf-bak
cat > /etc/httpd/conf.d/self-service-password.conf << EOF
<VirtualHost *>
    DocumentRoot /usr/share/self-service-password
    DirectoryIndex index.php
    AddDefaultCharset UTF-8
    Alias /ssp /usr/share/self-service-password
    <Directory "/usr/share/self-service-password">
        AllowOverride None
        Require all granted
    </Directory>
    LogLevel warn
    ErrorLog /var/log/httpd/ssp_error_log
    CustomLog /var/log/httpd/ssp_access_log combined
</VirtualHost>
EOF
```

参考官网配置：<https://ltb-project.org/documentation/self-service-password/1.2/config_apache>

2）修改Self Service Password的配置文件

```
vim /usr/share/self-service-password/conf/config.inc.php
#配置LDAP
$ldap_url = "ldaps://127.0.0.1:389";
$ldap_starttls = false;
$ldap_binddn = "cn=admint,dc=wmqe,dc=com";
$ldap_bindpw = "Admin123";
$ldap_base = "ou=People,dc=wmqe,dc=com";
$ldap_login_attribute = "uid";
$ldap_fullname_attribute = "cn";
$ldap_filter = "(&(objectClass=inetOrgPerson)($ldap_login_attribute={login}))";
$who_change_password = "manager";     #指定LDAP以什么用户身份更改密码
$keyphrase = "wmqe";

#配置邮件
$mail_from = "xxxxx@tenez.com";
$mail_from_name = "LDAP账号密码重置";
$mail_signature = "";          #mail签名
$notify_on_change = false;
$mail_sendmailpath = '/usr/sbin/sendmail';
$mail_protocol = 'smtp';
$mail_smtp_debug = 0;
$mail_debug_format = 'html';
$mail_smtp_host = 'smtp.qq.com';
$mail_smtp_auth = true;
$mail_smtp_user = 'xxxxx@qq.com';  #发送邮箱的账号
$mail_smtp_pass = 'xxxxxxx';      #发送邮箱的密码
$mail_smtp_port = 25;
$mail_smtp_timeout = 30;
$mail_smtp_keepalive = false;
$mail_smtp_secure = 'tls';
$mail_contenttype = 'text/plain';
$mail_wordwrap = 0;
$mail_charset = 'utf-8';
$mail_priority = 3;
$mail_newline = PHP_EOL;

#禁用问题验证
$use_questions=false;

#禁用短信验证
$use_sms= false;
```

参考官网配置：

配置ldap：<https://ltb-project.org/documentation/self-service-password/1.2/config_ldap>

配置邮箱：<https://ltb-project.org/documentation/self-service-password/1.2/config_mail>

### 7.3 重启httpd并访问

```
systemctl restart httpd
```

浏览器访问：http://192.168.159.130/

## 参考资料

> - []()
> - []()
