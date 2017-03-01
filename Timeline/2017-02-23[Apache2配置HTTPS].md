# Apache2 配置 HTTPS

*注：这里只提供Ubuntu方法，其他系统类比可得*

### 安装 `Apache2`

```
    apt-get update
    apt-get install apache2
```

### `Apache2` 配置目录

```
#       /etc/apache2/             //  Apache2根目录
#       |-- apache2.conf          //  Apache2配置文件
#       |-- conf-available        //  可用启配置目录（未开启）
#       |   `-- *.conf
#       |-- conf-enabled          //  已开启配置目录
#       |   `-- *.conf
#       |-- mods-available        //  可用模块目录（未开启）
#       |   |-- *.load
#       |   `-- *.conf
#       |-- mods-enabled          //  已开启模块配置目录
#       |   |-- *.load
#       |   `-- *.conf
#       |-- ports.conf            //  端口配置文件
#       |-- sites-available       //  可用虚拟主机vhost配置目录（未开启）
#       |   `-- *.conf
#       `-- sites-enabled         //  已开启虚拟主机vhost配置目录
#           `-- *.conf
```

### 生成自签证 `HTTPS`

创建根证书的私匙

```
  openssl genrsa -out xxxx.key 2048
```

利用私钥创建签名请求
```
  openssl req -new -subj "/C=CN/ST=Zhejiang/L=HzngZhou/O=Your Company Name/OU=xxxx.com/CN=xxxx.com" -key xxxx.key -out xxxx.csr
```

说明：这里/C表示国家(Country)，只能是国家字母缩写，如CN、US等；/ST表示州或者省(State/Provice)；/L表示城市或者地区(Locality)；/O表示组织名(Organization Name)；/OU其他显示内容，一般会显示在颁发者这栏。

将带口令的私钥移除

```
  mv xxxx.key xxxx.origin.key
  openssl rsa -in xxxx.origin.key -out xxxx.key
```

用Key签名证书

```
openssl x509 -req -days 3650 -in xxxx.csr -signkey xxxx.key -out xxxx.crt
```

为HTTPS准备的证书需要注意，创建的签名请求的CN必须与域名完全一致，否则无法通过浏览器验证

```
  xxxx.crt         自签名的证书  
  xxxx.csr         证书的请求  
  xxxx.key         不带口令的Key  
  xxxx.origin.key  带口令的Key
```

### 理解 数字证书（Certificate） crt, csr, key


1. 使用数字证书能够提高用户的可信度
2. 数字证书中的公钥，能够与服务端的私钥配对使用，实现数据传输过程中的加密和解密
3. 在证认使用者身份期间，使用者的敏感个人数据并不会被传输至证书持有者的网络系统上X.509证书包含三个文件：key，csr，crt。
    - key是服务器上的私钥文件，用于对发送给客户端数据的加密，以及对从客户端接收到数据的解密
    - csr是证书签名请求文件，用于提交给证书颁发机构（CA）对证书签名
    - crt是由证书颁发机构（CA）签名后的证书，或者是开发者自签名的证书，包含证书持有人的信息，持有人的公钥，以及签署者的签名等信息

备注：在密码学中，X.509是一个标准，规范了公开秘钥认证、证书吊销列表、授权凭证、凭证路径验证算法等。


### Apache SSL 配置文件说明

    SSLEngine on 启用SSL功能
    SSLCertificateFile 证书文件domain.crt
    SSLCertificateKeyFile 私钥文件domain.key
    SSLCertificateChainFile 证书链文件 CA.crt

### 配置Apache证书

    1. 打开SSL模块

    ```shell
      cd /etc/apache2/mods-enabled
      ln -s ../mods_mods-available/ssl.load
      ln -s ../mods_mods-available/ssl.conf
      ln -s ../mods_mods-available/socache_shmcb.load
    ```

    2. 把 `default-ssl.conf` 软链到 `sites-enabled`

    ```
      cd /etc/apache2/sites-enabled/
      ln -s ../sites-available/default-ssl.conf
    ```

    3. 新增或者打开配置

    ```
      SSLEngine on
      SSLCertificateFile 证书路径/xxxx.crt
      SSLCertificateKeyFile 证书路径/xxxx.key
    ```

### 使用 `CertBot` 生成信任证书

免费的HTTPS认证很多，最出名的为 **Let’s Encrypt** [官网](https://letsencrypt.org)

CertBot是什么？一个自动生成 Let’s Encrypt 工具（自动）

[Github](https://github.com/certbot/certbot) [官网](https://certbot.eff.org)

知乎一篇温江讲述了另一种方法安装和使用 CertBot [直达链接](https://zhuanlan.zhihu.com/p/21286171)

官网有教程，这里就不详细描述安装过程。

腾讯，阿里，都有免费证书可以使用，级别为EV

### DV，OV，EV

目前主流的SSL证书主要分为DV SSL 、 OV SSL 、EV SSL。

  - DV SSL

    >DV SSL证书是只验证网站域名所有权的简易型（Class 1级）SSL证书，可10分钟快速颁发，能起到加密传输的作用，但无法向用户证明网站的真实身份。
    目前市面上的免费证书都是这个类型的，只是提供了对数据的加密，但是对提供证书的个人和机构的身份不做验证。

  - OV SSL

    >OV SSL,提供加密功能,对申请者做严格的身份审核验证,提供可信身份证明。
    和DV SSL的区别在于，OV SSL 提供了对个人或者机构的审核，能确认对方的身份，安全性更高。
    所以这部分的证书申请是收费的~

  - EV SSL
    >超安=EV=最安全、最严格 超安EV SSL证书遵循全球统一的严格身份验证标准，是目前业界安全级别最高的顶级 (Class 4级)SSL证书。
    金融证券、银行、第三方支付、网上商城等，重点强调网站安全、企业可信形象的网站，涉及交易支付、客户隐私信息和账号密码的传输。
    这部分的验证要求最高，申请费用也是最贵的。

### `HTTP` 重定向 `HTTPS`

```shell
    RewriteEngine on
    RewriteCond %{SERVER_PORT} !^443$
    RewriteRule ^.*$ https://%{SERVER_NAME}%{REQUEST_URI} [L,R=301]
```

重定向不能完全保证安全问题，所以需要使用严格安全传输 `HSTS`

### 严格传输安全 `HSTS`

引用资源: [Mozilla Developer Network](https://developer.mozilla.org/zh-CN/docs/Security/HTTP_Strict_Transport_Security)

>如果一个 web 服务器支持 HTTP 访问，并将其重定向到 HTTPS 访问的话，那么访问者在重定向前的初始会话是非加密的。举个例子，比如访问者输入 http://www.foo.com/ 或直接输入 foo.com 时。
这就给了中间人攻击的一个机会，重定向可能会被破坏，从而定向到一个恶意站点而不是应该访问的加密页面。
HTTP 严格传输安全（HSTS）功能使 Web 服务器告知浏览器绝不使用 HTTP 访问，在浏览器端自动将所有到该站点的 HTTP 访问替换为 HTTPS 访问。

各种服务器配置参考：[HTTP Strict Transport Security for Apache, NGINX and Lighttpd](https://raymii.org/s/tutorials/HTTP_Strict_Transport_Security_for_Apache_NGINX_and_Lighttpd.html)

  1.    添加扩展 `mod_headers.so`&`mod_rewrite.so`
  ```shell
    cd /etc/apache2/mods-enabled

    ln -s ../mods_mods-available/rewrite.load

    ln -s ../mods_mods-available/headers.load
  ```

  2.    添加 `Header` （修改SSL虚拟主机）
  ```bash
    <VirtualHost 192.168.1.1:443>
        Header always set Strict-Transport-Security “max-age=31536000; includeSubDomains”
    </VirtualHost>
  ```

  3.    添加重定向 修改HTTP虚拟主机
  ```bash
      RewriteEngine On
      RewriteCond %{HTTPS} off
      RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
  ```
  OR
  ```bash
      RewriteEngine on
      RewriteCond %{SERVER_PORT} !^443$
      RewriteRule ^.*$ https://%{SERVER_NAME}%{REQUEST_URI} [L,R=301]
  ```


# 附录

### HTTPS相关网站

1. [https://www.geocerts.com/ssl_checker](https://www.geocerts.com/ssl_checker) --判断SSL证书错误点  

2. [https://www.ssllabs.com](https://www.ssllabs.com/) --HTTPS公认评分网站

### Linux链接

1. 软连接
  * 软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式  
  * 软链接可以 跨文件系统 ，硬链接不可以     
  * 软链接可以对一个不存在的文件名进行链接     
  * 软链接可以对目录进行链接  

2. 硬链接
  * 硬链接，以文件副本的形式存在。但不占用实际空间。
  * 不允许给目录创建硬链接
  * 硬链接只有在同一个文件系统中才能创建

命令方式: `ln [参数][源文件或目录][目标文件或目录]`

必要参数:
```
    -b 删除，覆盖以前建立的链接
    -d 允许超级用户制作目录的硬链接
    -f 强制执行
    -i 交互模式，文件存在则提示用户是否覆盖
    -n 把符号链接视为一般目录
    -s 软链接(符号链接)
    -v 显示详细的处理过程
```

所以使用软链接方式可以为:   

`ln -s ../mods_mods-available/rewrite.load`     
目标文件被省略为，当前所在目录，文件名为文件名     
使用 `ll` OR `ls -l` 可查看到链接的位置
