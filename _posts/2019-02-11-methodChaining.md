---
title: "JavaScript methodChaining"
date: 2019-02-11 18:00:00 -0400
categories: JavaScript
---
# Method Chaining


```javascript
//////////////////////////
// WITHOUT METHOD CHAINING

var $div = $('#my-div');         // assign to var

$div.css('background', 'blue');  // set BG
$div.height(100);                // set height
$div.fadeIn(200);                // show element

```
객체를 만들어 함수를 하나씩 불러주던 기존 방식에서

```javascript
///////////////////////
// WITH METHOD CHAINING

$('#my-div').css('background', 'blue').height(100).fadeIn(200);
```
메서드를 순차적으로 부르는 방식으로 구현하는것.
명백히 의미있는 반환값을 가지지 않는다면 항상 this를 반환한다.

## 장점
* 함수들이 순차적으로 불리는 것을 볼수있다.
* 코드량을 줄일 수 있다.
* 하나의 문장처럼 읽히게 할 수 있다.
* 유지보수가 개선된다.

## 단점
* 디버깅이 어렵다. 
(에러발생 라인을 찾을 수는 있어도, 해당 라인에서 여러일을 수행하고있을 수 있음. - train wreck [clean code에서는 줄단위로 나누라고 말한다.])
  
Jquery 라이브러리 등에서도 널리 사용되고 있으며,
DOM API에서도 일부 체이닝 패턴을 사용하는 경향이 있다.


# Prototype vs Class 

## protoType
prototype을 이용해서 클래스 와 비슷한 역할을 할 수 있다.

```javascript
    var UserInfo = function(){  
        this._user = null;
        this._password = null;
    };

    UserInfo.prototype.user = function(user){  
        this._user = user;
        return this;
    }

    UserInfo.prototype.password = function(password){  
        this._password = password;
        return this;
    };

    UserInfo.prototype.print = function(){
        console.log(this._user);
        console.log(this._password);
    }

    new UserInfo.user("kim").password("123456").print();

```

prototype을 사용하면 UserInfo.prototype 이라는 인스턴스가 만들어져있고, 
UserInfo에서 파생된 어떤 객체든 해당 값, 함수를 가져다쓸수있다.

- 모든 객체는 __proto__라는 prototype Object와 constructor을 가진다.
- prototype Object는 일반적인 객체와 동일하게 속성이나 메소드를 삭제하거나 변경 추가 할 수 있다.
- 

### 프로토타입 체인
- __proto__는 객체가 생성될 때 조상이었던 함수의  Prototype Object를 가르킨다.
- 상위 프로토타입에서 정의를 찾지 못했을 경우 최상위 Object 객체의 prototype Object 까지 도달하여 못찾으면 undefined를 리턴한다. (ex) Object 객체의 toString() )
- class의 상속처럼 동작한다.

## Class

ES6 부터 class 키워드가 있다.
OOP언어처럼 class, constructor, super, extents도 사용가능하다.

- class에서의 extends는 사실상 prototype chain과 동일하게 동작한다.
- class에서의 method들은 protoType Object의 method와 동일하다.

class키워드는 있으나, 동작은 prototype과 동일하다.
class 키워드로 구현을 좀더 친숙하고 간단하게 할수있는 차이.

```javascript
    class UserInfo {  
        constructor(user,password){
            this._user = user;
            this._password = password;
        }
        
        user() {
            this._user = user;
            return this;
        }

        password () {
            this._password = password;
            return this; 
        }

        print() {
            console.log(this._user);
            console.log(this._password);
        }
    };

    new UserInfo.user("kim").password("123456").print();
```


[1]  js prototype : https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67
[2] prototype vs class : https://medium.com/@parsyval/javascript-prototype-vs-class-a7015d5473b
[3] method Chain :  https://schier.co/blog/2013/11/14/method-chaining-in-javascript.html
[4] chaining pattern : https://webclub.tistory.com/528
