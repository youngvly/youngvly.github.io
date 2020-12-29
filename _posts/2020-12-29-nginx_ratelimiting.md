---
title: "Nginx RateLimitting"
date: 2020-12-29 21:57:00 -0400
categories: ETC
tags: nginx
---
# RateLimitting

[NGINX Rate Limiting](https://www.nginx.com/blog/rate-limiting-nginx/)

- Zone  : 각 ip별 상태와, request-limited가 걸린 url에 얼마나 자주 요청이 들어오는지 등을 저장하는 공유 저장소를 사용을 정의할수있다. 
zone=<zone_name>:<zone:size>

```bash
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
 
server {
    location /login/ {
        limit_req zone=mylimit;
        
        proxy_pass http://my_upstream;
    }
}
```