---
title: "HTML DOM에 Element 넣는 방법 비교 (String vs elementNode)"
date: 2019-01-14 23:00:00 -0400
categories: JavaScript
---

# Html DOM에 태그 넣는 방법 비교 

1. innerHTML
```javascript
document.getElementById('container').innerHTML += '<p>Just some <span>text</span> here</p>';
```

2. jQuery
```javascript
$('#container').append('<p>Just some <span>text</span> here</p>');
```

3. appendChild
```javascript
var div = document.createElement("div");
div.innerHTML = '<p>Just some <span>text</span> here</p>';
document.getElementById('container').appendChild(div)
```

4. insertAdjacentHTML
```javascript
document.getElementById('container').insertAdjacentHTML('beforeend', '<p>Just some <span>text</span> here</p>');
```

<img src="https://i.stack.imgur.com/YD0Wq.png"/>
*Set up environment (2018.07.02) MacOs High Sierra 10.13.3 on Chrome 67.0.3396.99 (64-bit), Safari 11.0.3 (13604.5.6), Firefox 59.0.2 (64-bit) )*

**순위(빠른순)** : insertAdjacentHTML > appendChild > jquery > innerHTML

+ performance diffrence depends on browser.
+ IE 에서는 innerHTML > Dom Function 으로 빨랐다.

## 결론 
DOM에 elements를 추가하는 방법들을 모두 속도를 측정하여 비교하였다.
각 방법별 속도차이는 브라우저에 따라 다를 수 있어 무엇이 좋다고 단정지을 수 없는 것 같다.
string을 직접 넣는 방법은 보기에는 깔끔하나 비교적 string을 합치거나 일부를 수정할때에 비효율적일 수 있다.
elements를 생성하여 넣는 방법은 비교적 빠르고, 깊이가 깊은 노드를 생성하기에 적합하다.

**stack overflow의 한 사용자가 정의한 사용방법**
+ minor changes > innerHTML : 문구 수정,링크 추가 등의 간단한 수정
+ major changes > appendChild(createElement()) : 한개의 컨테이너에 많은 컴포넌트를 넣을경우, 노드의 깊이가 깊을경우.(2 이상)
+ table의 row를 추가한다고 할때, 적은 수의 row를 추가하는것은 string을 넣는것이 나을 수 있으나, 다수의 row를 추가할 때에는 appendChild를 사용하는것이 좋다.

[1] performance Test : https://stackoverflow.com/questions/7327056/appending-html-string-to-the-dom/7327125 
[2] adding elements as a string vs createElement() : https://stackoverflow.com/questions/5878281/adding-elements-as-a-string-vs-createelement 
[3] fast way to append elements to DOM : https://howchoo.com/g/mmu0nguznjg/learn-the-slow-and-fast-way-to-append-elements-to-the-dom 