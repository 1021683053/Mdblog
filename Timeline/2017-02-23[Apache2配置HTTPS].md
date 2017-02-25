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

### 签证配置文件说明

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


### `HTTP` 重定向 `HTTPS`

### 使用 `CertBot` 生成信任证书

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
