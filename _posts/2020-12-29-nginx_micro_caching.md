---
title: "Nginx MicroCacheing"
date: 2020-12-29 21:57:00 -0400
categories: ETC
tags: nginx
---
# MicroCacheing

```json
http {

proxy_cache_path 
	/tmp/cache  # 여기에 캐시저장
	keys_zone=zone_name:10m # 설정한 zone 이름, 사이즈
	levels=1:2 # 
	inactive=600s; # 600초동안 cache hit 없으면 캐시삭제

server {
	location / {
		proxy_pass **;
		proxy_cache zone_name;
		
	}
}
}
```

proxy_cache_valid는 http{}, server{}, location{} 에서 사용될 수 있다.

[Module ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html?&_ga=2.24325823.424123966.1594014515-215263839.1582249796#proxy_cache_purge)

- proxy_cache zone 이름 선택
- proxy_cache_purge
    - example
    request Method가 PURGE면 모든 캐시를 삭제한다.

    ```json
    map $request_method $purge_method {
        PURGE   1;
        default 0;
    }

    server {
        ...
        location / {
            proxy_pass http://backend;
            proxy_cache cache_zone;
            proxy_cache_key $scheme$proxy_host$request_uri; //default_value
            proxy_cache_purge $purge_method;
        }
    }
    ```

    - 1.7.12 이상.
- proxy_cache_revalidate
- proxy_cache_use_stale : 어떤 상황에서 expired된 캐시 사용 여부
    - updating | error: 캐시 업데이트시, 업스트림 에러시.
    - proxy_next_upstream 와 같은 파라미터로 사용되어야함.

- proxy_cache_lock 동일 proxy_cache_key 를 생성하는 요청에 대해 최초 요청만 허용. 나머지는 대기
- proxy_cache_lock_timeout 타임아웃이 지나면 나머지 요청도 프록시서버로 전달, 응답은 캐싱된다.
- proxy_cache_key 캐시키
- proxy_cache_valid 응답 코드에 따라 캐싱시간을 다르게 설정가능.
- proxy_cache_methods 특정 메서드만 허용
- proxy_cache_max_range_offset request 사이즈가 넘으면 캐싱되지 않음.
- proxy_cache_min_uses n번 이상 요청시에만 캐싱.
- proxy_cache_path
    - levels 1:2 : 파일이 2뎁스까지 지정되어 저장됨. 
    ex) /data/nginx/cache/c/29/12345678
    - 캐싱이 되기전에 temp파일이 지정되는데, 별도 지정없으면 동일 폴더에 저장됨. copy보다는 rename이 간단하므로, 동일폴더 저장 지향
    - One megabyte zone can store about 8 thousand keys.
    - inactive 시간 이후 캐시 자동삭제, 디폴트는 10분
    - cache manager
        - min_free 최소한으로 남겨둬야할 파일시스템의 여유공간,
        max_size 저장될 수 있는 캐시의 최대크기,
        넘으면 사용안된순서로 자동삭제 iteration이 돈다.
        - manager_threshold 1번의 iteration이 도는 시간, 디폴트 200ms
        - manager_files 1번의 iteration에 max로 삭제될 파일 수, 디폴트 100
        - manager_sleep 1번의 iteration 이후 쉬는 시간 , 디폴트 50ms
    - cache loader
        - 한번의 iteration동안 loader_files 이상의 파일이 캐싱되지 못한다., 디폴트 100
        - loader_threshold, loader_sleep , cache manager와 동일

- proxy_cache_background_update 캐싱 업데이트를 background로


### check needed

- [https://brunch.co.kr/@alden/26](https://brunch.co.kr/@alden/26) 에 따르면 캐싱 적용시 요청이 들어와도 같은 timestamp가 찍힘.