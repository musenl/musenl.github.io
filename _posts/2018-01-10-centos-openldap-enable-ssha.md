---
layout: post
title: Centos6.5下OpenLdap搭建(环境配置+双机主从配置+LDAPS+enable SSHA)
categories: 安全运维
description: 在Centos6.5环境下搭建OpenLdap(环境配置+双机主从配置+LDAPS+enable SSHA)
keywords: Centos6.5,OpenLdap,SSHA
---

OpenLDAP 是 LDAP 协议的一个开源实现。LDAP 服务器本质上是一个为只读访问而优化的非关系型数据库。它主要用做地址簿查询（如 email 客户端）或对各种服务访问做后台认证以及用户数据权限管控。（例如，访问 Samba 时，LDAP 可以起到域控制器的作用；或者 [Linux 系统认证](https://wiki.archlinux.org/index.php/LDAP_authentication) 时代替 `/etc/passwd` 的作用。）

## 为什么要做这个事

公司打算做统一认证，由于LDAP支持radius，可以把网络、安全设备集中在一个LDAP中认证；

## 环境

Centos 6.5 双机：

>  10.65.0.38
>
>  10.65.0.39
>

## 安装

centos6.4默认安装了LDAP，但没有装ldap-server和ldap-client， yum安装最简单：

 ```
 su root
 # yum install -y openldap openldap-servers openldap-clients
 ```

准备数据库目录，openldap配置文件

 ```
# cp /usr/share/openldap-servers/slapd.conf.obsolete /etc/openldap/slapd.conf
# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
# mv /etc/openldap/slapd.d{,.bak}
 ```

为什么要将slapd.d目录移动为.bak文件，如果这个文件存在，直接修改slapd.conf后还需要重新生成slapd.d目录，因为官方 对于 OpenLDAP 2.4 ，不推荐使用 `slapd.conf` 作为配置文件。从这个版本开始所有配置数据都保存在 `/etc/openldap/slapd.d/`中。如果没有以上第三步的操作，每次重新生成slapd.d的步骤为：

 ```
# slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d/
 ```

 如果没有将slapd.d改变名称，意味着每次改变slapd.conf时，都需要运行以上命令，生成新的slapd.d目录，接着要改变slapd.d的属主：

 ```
# chown -R ldap:ldap /etc/openldap/slapd.d
 ```

 ```
# slapindex
# chown ldap:ldap /var/lib/openldap/openldap-data/*
 ```

 最后，在将openLDAP服务重启，保证配置文件变更生效，这里建议直接就吧slapd.d目录做个备份，然后删掉，这样改变slapd.conf后，直接重启服务就可生效。

 ```
# service slapd restart
 ```


## 配置

 vim修改slapd.conf文件，步骤如下：

 1、设置目录树后缀，一般以公司域名来设置

 2、设置LDAP管理员DN

 3、修改LDAP管理员口令

 找到语句：

 ```
suffix "dc=my-domain,dc=com"
 ```

 ```
rootdn "cn=Manager,dc=my-domain,dc=com"
 ```

 ```
rootpw secret
 ```

 将其改为：

 ```
suffix "dc=example,dc=com"
 ```

 ```
rootdn "cn=Manager,dc=example,dc=com"
 ```

 ```
rootpw {SSHA}NXV9Fl28qCHMmA6P sjhVX0uejTKE6OYr
 ```

 以上的密文值是通过以下命令生成的

 ```
# slappasswd -s your_secret_string
 ```

 配置文件修改之后分下权限

 ```
chown ldap.ldap /etc/openldap/*
chown ldap.ldap /var/lib/ldap/*
 ```


### 重启服务：

 ```
 # service slapd restart
 ```

### 数据导入

新建文件example.ldif

```
dn:dc=example,dc=com
objectclass:dcObject
objectclass:organization
o:Example, Inc.
dc:example

dn:cn=Manager,dc=example,dc=com
objectclass:organizationalRole
cn:Manager
```

### 通过命令导入：

 ```
 /usr/bin/ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f example.ldif
 ```

 

### 配置基于TLS的OpenLDAP

 如果要使用基于TLS的LDAPS安全协议来连接，必须先生成证书，可以使用自签发的SSL证书。说到这个自签发证书，被狠狠的坑了很久，查了无数文档，直到看到wiki上的一句话才解决问题，把wiki上的官方文档贴一下，让大家少进坑为妙：

 ```
 Warning: OpenLDAP cannot use a certificate that has a password associated to it.
 ```

 如果OpenLDAP要利用证书来使用LDAPS协议，证书必须不含密码。

 创建的步骤为：

 ```
 $ openssl req -new -x509 -nodes -out slapdcert.pem -keyout slapdkey.pem -days 365
 ```

 接下来将会要求输入一些，证书创建的信息，包括省份，城市，之类的，按照实际的填即可，有些可以为空，值得注意的是，CN值必须为服务器主机名或者IP值

 CN (Common Name)：10.65.0.38

 将生成的证书文件 slapdcert.pem及私钥文件slapdkey.pem，移动到/etc/openldap/openldap/ssl目录下（没有的话可以先创建）：

 ```
 # mv slapdcert.pem slapdkey.pem /etc/openldap/ssl/
 # chmod -R 755 /etc/openldap/ssl/
 # chmod 400 /etc/openldap/ssl/slapdkey.pem
 # chmod 444 /etc/openldap/ssl/slapdcert.pem
 # chown ldap /etc/openldap/ssl/slapdkey.pem
 ```

### 配置基于SSL的slapd

 修改配置文件（/etc/openldap/slapd.conf）

 ```
 # Certificate/SSL Section
 TLSCipherSuite HIGH:MEDIUM:-SSLv2
 TLSCertificateFile /etc/openldap/ssl/slapdcert.pem
 TLSCertificateKeyFile /etc/openldap/ssl/slapdkey.pem
 ```

 关闭openldap，然后重新启动基于SSL的slapd

 ```
 /etc/init.d/slapd stop
 ```

 ```
 slapd -h "ldap:/// ldaps:///"
 ```

### 配置双机主从复制 LDAP：

 按照以上步骤安装从OpenLDAP服务器，使用Syncrepl方式来同步主从服务器数据，该方式是slave服务器以拉的方式同步master的用户数据，  该方式缺点：当你修改一个条目中的一个属性值（or大批量的万级别的某1属性值），它不是简单的同步过来这些属性，而是把修改的条目一起同步更新来。


1、配置master LDAP , vim /etc/openldap/slapd.conf 加入以下

```
#replication
index entryCSN,entryUUID eq  
overlay syncprov  
syncprov-checkpoint 100 10  
syncprov-sessionlog 100
```

2、配置 slave LDAP ，vim /etc/openldap/slapd.conf 加入以下

```
#replication
index entryCSN,entryUUID eq

syncrepl rid=123
        provider=ldap://10.65.0.38
        type=refreshOnly
        interval=00:00:00:00
        searchbase="dc=example,dc=com"
        filter="(objectClass=inetOrgPerson)"
        scope=sub                   
        attrs="cn,sn,ou,mail,sambaNTPassword,sambaSID,uid,userPassword"
        schemachecking=off
        bindmethod=simple
        binddn="cn=Manager,dc=example,dc=com"
        credentials=your_secret
```

3、配置导入明文密码是自动变为ssha hash值

vim /etc/openldap/slapd.conf ,将以下值加入(1、加入这个schema 2、启动ppolicy模块 3、启用明文转换SSHA)

```
#Include schema

include /usr/local/etc/openldap/schema/ppolicy.schema

# Load dynamic backend modules:

moduleload ppolicy.la

# After database definitions, You can add followings.

overlay ppolicy
ppolicy_hash_cleartext
```

### 重启LDAP服务器

测试：

创建一个LDIF文件

```
dn: cn=john,ou=Users,dc=example,dc=com
objectClass: person
sn: doe
cn: john
userPassword: johnldap
```

使用ldapadd命令导入：

 ```
 ldapadd -x -D "cn=asela,dc=example,dc=com" -W -f user.ldif
 ```

*使用客户端打开该dn值，将会看到密码已经变为\**SSHA的密码。***

**PS：推荐使用的客户端LDAP administrator**

这个直接去官网下载 <http://www.ldapbrowser.com/download.htm>

配置连接:

![image](https://raw.githubusercontent.com/musenl/musenl.github.io/master/images/posts/ldap/658832-20160628192245093-654773812.png)

![image](https://raw.githubusercontent.com/musenl/musenl.github.io/master/images/posts/ldap/658832-20160628192247327-1199690489.png)

连接后的效果

![image](https://raw.githubusercontent.com/musenl/musenl.github.io/master/images/posts/ldap/658832-20160628192248952-999954053.png)

 

参考的blog：

1、<http://my.oschina.net/5lei/blog/193484>
2、http://xacmlinfo.org/2015/06/25/enable-hash-passwords-in-openldap/
3、https://wiki.archlinux.org/index.php/OpenLDAP_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
4、http://407711169.blog.51cto.com/6616996/1529506