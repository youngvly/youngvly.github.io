---
title: "send ajax-JsonArray and recieve with @RequestBody"
date: 2019-01-12 22:37:00 -0400
categories: Spring-boot-study
---
ajax 로 post요청을 보내는중, jsonArray를 보내는 과정에서, 서버가 배열의 크기만큼의 데이터를 받기는 하지만 실제 값은 null과 0 인 default값으로 값이들어온다.


## 오류상황
**보내지는 Json객체**
```json
{
    "param":[
        {
            "name":"철수",
            "age":19
        },{
            "name":"민수",
            "age":20
        }
    ]
}
```

ajax 요청 스크립트
```javascript
$.ajax({
            data : JSON.stringify(data)),
            url: "/ajax/test/post",
            type: "POST",
            contentType:"application/json"
        }).done(function (data) {});
```

```java
//Domain
public class ExParam {

   private static class Some{
        private String sss;
        private int iii;
   }

   private Collection<Some> param ;
}
//Controller
 public void ajaxTestPostController (
            @RequestBody ExParam exparam){};
```


## 수정뒤
**보내지는 Json객체**
```json
[
    {
        "name":"철수",
        "age":19
    },{
        "name":"민수",
        "age":20
    }
]

```

ajax 요청 스크립트
```javascript
$.ajax({
            data : JSON.stringify(array)),
            url: "/ajax/test/post",
            type: "POST",
            contentType:"application/json"
        }).done(function (data) {});
```

```java
//Domain
public class ExParam {
    private String sss;
    private int iii;
}
//Controller
 public void ajaxTestPostController (
            @RequestBody Collection<ExParam> exparam){};
```



