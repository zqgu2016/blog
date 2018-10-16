## [Let's Encrypt](https://letsencrypt.org/)

#### Let's Encrypt是一个免费的、自动化的、开放的证书颁发机构。

### 1、克隆申请工具acme-tiny

```shell
git clone https://github.com/diafygi/acme-tiny.git
cd acme-tiny
```

### 2、创建Let's Encrypt账户的私钥

```shell
openssl genrsa 4096 > account.key
```

### 3、创建域名的证书请求文件(CSR)

```shell
openssl genrsa 4096 > domain.key

#单个域名
openssl req -new -sha256 -key domain.key -subj "/CN=yoursite.com" > domain.csr

#多个域名
openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:yoursite.com,DNS:www.yoursite.com")) > domain.csr
```

### 3、新建challenge文件夹

```shell
mkdir -p /var/www/challenges/
```

```nginx
# Example for nginx
server {
    listen 80;
    server_name yoursite.com www.yoursite.com;

    location /.well-known/acme-challenge/ {
        alias /var/www/challenges/;
        try_files $uri =404;
    }

    ...the rest of your config
}
```
`注： Let's Encrypt将向你服务器上的80端口发送一个http请求，所以必须保证challenge文件夹可访问`

### 4、申请证书

```shell
python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /var/www/challenges/ > ./signed_chain.crt
```

### 5、安装证书

```nginx
server {
    listen 443 ssl;
    server_name yoursite.com, www.yoursite.com;

    ssl_certificate /path/to/signed_chain.crt;
    ssl_certificate_key /path/to/domain.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA;
    ssl_session_cache shared:SSL:50m;
    ssl_dhparam /path/to/server.dhparam;
    ssl_prefer_server_ciphers on;

    ...the rest of your config
}

server {
    listen 80;
    server_name yoursite.com, www.yoursite.com;

    location /.well-known/acme-challenge/ {
        alias /var/www/challenges/;
        try_files $uri =404;
    }

    ...the rest of your config
}
```

### 定时更新证书

`创建renew_cert.sh文件`

```shell
touch renew_cert.sh
```

`在renew_cert.sh中添加自动更新脚本`

```shell
#!/usr/bin/sh
python /path/to/acme_tiny.py --account-key /path/to/account.key --csr /path/to/domain.csr --acme-dir /var/www/challenges/ > /path/to/signed_chain.crt || exit
service nginx reload
```

```shell
# 每个月1号执行一次
0 0 1 * * /path/to/renew_cert.sh 2>> /var/log/acme_tiny.log
```

### 注意事项

- 安装很更新过程过中需要注意文件的读写问题
- 每次申请的证书有效期是3个月
- 一个ip3小时内最多申请10个证书, 一个根域名，一个礼拜最多可申请50个证书，[详见](https://letsencrypt.org/docs/rate-limits/)

#### 参考

- https://github.com/diafygi/acme-tiny