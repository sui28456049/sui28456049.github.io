---
title: CA服务器和OpenSSL自签证书
date: 2018-11-28 16:45:13
tags: Linux
category: Linux
---
>>>非对称加密,CA 也拥有一个证书（内含公钥和私钥）。用户通过验证 CA 的签字从而信任 CA ，任何人都可以得到 CA 的证书（含公钥），用以验证它所签发的证书。如果用户想得到一份属于自己的证书，他应先向 CA 提出申请。在 CA 判明申请者的身份后，便为他分配一个公钥，并且 CA 将该公钥与申请者的身份信息绑在一起，并为之签字后，便形成证书发给申请者。如果一个用户想鉴别另一个证书的真伪，他就用 CA 的公钥对那个证书上的签字进行验证，一旦验证通过，该证书就被认为是有效的。证书实际是由证书签证机关（CA）签发的对用户的公钥的认证

# 建立CA服务器(47.98.233.59)

## openssl配置

centos与CA配置的相关文件是/etc/pki/tls/openssl.cnf， 里面默认定义了各类功能文件的名字和所在的目录

```bash
 [ CA_default ]
 41
 42 dir     = /etc/pki/CA       # Where everything is kept
 43 certs       = $dir/certs        # Where the issued certs are kept
 44 crl_dir     = $dir/crl      # Where the issued crl are kept
 45 database    = $dir/index.txt    # database index file.
 46 #unique_subject = no            # Set to 'no' to allow creation of
 47                     # several ctificates with same subject.
 48 new_certs_dir   = $dir/newcerts     # default place for new certs.
 49
 50 certificate = $dir/cacert.pem   # The CA certificate
 51 serial      = $dir/serial       # The current serial number
 52 crlnumber   = $dir/crlnumber    # the current crl number
 53                     # must be commented out to leave a V1 CRL
 54 crl     = $dir/crl.pem      # The current CRL
 55 private_key = $dir/private/cakey.pem# The private key
 56 RANDFILE    = $dir/private/.rand    # private random number file
 57
 58 x509_extensions = usr_cert      # The extentions to add to the cert
 59
 60 # Comment out the following two lines for the "traditional"
 61 # (and highly broken) format.
 62 name_opt    = ca_default        # Subject Name options
 63 cert_opt    = ca_default        # Certificate field options
```
## 将一台主机(47.98.233.59)配置成具有签证能力的CA

1、在/etc/pki/CA/目录下创建所需文件index.txt和serial，是记录签证相关的信息的。

```bash
cd /etc/pki/CA
touch index.txt
echo 01 >serial
```
2、在/etc/pki/CA/目录下，生成私钥并保存在private/cakey.pem文件中保存。

```bash
openssl genrsa -out private/cakey.pem 2048
```
3、在CA主机使用如下命令自签证书，使这个主机成为具有签证能力的CA，命令的参数说明：-new说明是签发新证书，-x509用于自签，-key指明私钥的文件，-days指的是证书的有效天数，-out表示证书输出的文件。

```bash
openssl req -new -x509 -key private/cakey.pem -days 360 -out cacert.pem
```


You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanxi
Locality Name (eg, city) [Default City]:yuncheng
Organization Name (eg, company) [Default Company Ltd]:sui.inc
Organizational Unit Name (eg, section) []:sui
Common Name (eg, your name or your server's hostname) []:sui.com
Email Address []:28456049@qq.com

## 请求主机(121.65.22.128)请求具有签证能力的主机(47.98.233.59)签发签证

1、在/etc/httpd/ssl/目录下生成私钥，并保存在httpd.key中

```bash
openssl genrsa -out sui.key 2048
```

2、使用httpd.key私钥生成未签证书，并保存在httpd.csr中。注意：无需 -x509 参数

```bash
openssl req -new -key sui.key -days 365 -out sui.csr
```

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanxi
Locality Name (eg, city) [Default City]:yuncheng
Organization Name (eg, company) [Default Company Ltd]:sui.inc
Organizational Unit Name (eg, section) []:sui
Common Name (eg, your name or your server's hostname) []:sui.com
Email Address []:28456049@qq.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

3、将未签证书httpd.csr复制到（CA）主机中。

```bash
scp httpd.csr root@47.98.233.59:/tmp/httpd.csr
root@47.98.233.59's password:
httpd.csr                                     100% 1037     1.0KB/s   00:00
```
4、在(CA服务器里47.98.233.59)使用如下命令对未签证书httpd.csr进行签证，并保存在httpd.crt中。

```bash
openssl ca -in /tmp/httpd.csr -out /tmp/httpd.crt -days 365
```

## 使用证实 nginx 配置ssl

```bash
server {
 		server_name sui.com;
        listen [::]:443 ssl default_server;

        ssl on;
        ssl_certificate /user/local/nginx/ssl/sui.crt;
        ssl_certificate_key /usr/local/nginx/ssl/sui.key;
}
```

浏览器提示此网站的安全证书不安全

CA根证书加入到浏览器


# 自签证书

## 方式一

通过openssl生成私钥

```bash
openssl genrsa -out server.key 2048
```
使用私钥生成自签名的cert证书文件，以下是通过参数只定证书需要的信息

```bash
openssl req -new -x509 -days 3650 -key server.key -out server.crt -subj "/C=CN/ST=mykey/L=mykey/O=mykey/OU=mykey/CN=domain1/CN=domain2/CN=domain3"
```

## 方式二(推荐)

通过openssl生成私钥

```bash
openssl genrsa -out server.key 2048
```

根据私钥生成证书申请文件csr

```bash
openssl req -new -key server.key -out server.csr
```

这里根据命令行向导来进行信息输入：

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanxi
Locality Name (eg, city) [Default City]:yuncheng
Organization Name (eg, company) [Default Company Ltd]:sui.inc
Organizational Unit Name (eg, section) []:sui
Common Name (eg, your name or your server's hostname) []:sui.com
Email Address []:28456049@qq.com

通配域名生成: Common Name输入：*.yourdomain.com，这种方式生成通配符域名证书

使用私钥对证书申请进行签名从而生成证书:

```bash
openssl x509 -req -in server.csr -out server.crt -signkey server.key -days 3650
```

这样就生成了有效期为：10年的证书文件，对于自己内网服务使用足够。

## 方式三

直接生成证书文件

```bash
openssl req -new -x509 -keyout server.key -out server.crt -config openssl.cnf
```


