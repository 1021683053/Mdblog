# Apache2 配置 HTTPS

*注：这里只提供Ubuntu方法，其他系统类比可得*

### 安装 `Apache2`

```
    apt-get update
    apt-get install apache2
```

### `Apache2` 配置目录

### 生成自签证 `HTTPS`

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
