## 源码安装nginx

---

```shell
// 安装依赖库
yum install zlib-devel -y
// 下载
wget http://nginx.org/download/nginx-1.14.2.tar.gz
tar -zxvf nginx-1.14.2.tar.gz
// 进入解压目录
cd nginx-1.14.2
// 配置(由于rewrite模块需要PCRE库，略去)
./configure --without-http_rewrite_module
// 编译安装
make && make install
// 添加软链
ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
// 启动
nginx -v
```
