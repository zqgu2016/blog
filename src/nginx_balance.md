## 负载均衡及健康检查

---

```nginx
upstream backend {
    server 10.26.254.155:8083 max_fails=1 fail_timeout=100s weight=3;
    server 10.28.13.106:8083 max_fails=1 fail_timeout=100s weight=7;
    check interval=3000 rise=2 fall=5 timeout=2000 default_down=false type=http;
    check_http_send "GET /travel_web/system/heartbeat HTTP/1.0\r\n\r\n";
}

location / {
    proxy_pass http://backend/travel_web;
}

location /status {
    check_status;
    access_log off;
}
```
