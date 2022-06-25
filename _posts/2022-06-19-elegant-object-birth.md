---
layout: post
title: "ElegantObject 01 Birth"
date: 2022-06-19 23:51
categories: CleanCode
tags: cleanCode, elegantObject
---
# 01 Birth
### -er로 끝나는 이름을 사용하지말자.
> 흠... 클래스의 크기를 줄이기위해 목적이 드러나는 이름으로 클래스가 분리되는 경우는? 안티인가
> DateTimeFormatter -> ?? DateTimeFormat? 나쁘지않긴한듯 하다
- 클래스는 객체의 팩토리이다.
- 클래스가 무엇을 하고있는지, 기능에 기반에서 이름을 짓는 방법은 지양해야한다.
- What class does (x) -> What class is (O)
- anti ex) Controller, Helper, Encoder,,, 
    - Target, EncodedText, 
- er 이 붙지만 그자체로 명사인경우는 괜찮다. ex) User, Computer
- er로 끝는 경우, 이 클래스는  어떤 데이터를 다루는 절차들의 집합일 뿐이다.
- Utility class도 별로인데, 이유는 3.2.3에서 보자
### 생성자 하나를 주 생성자로 만들자.
- 응집도가 높고 견고한 클래스에는 적은수의 메서드와 상대적으로 더 많은수의 생성자가 존재한다.
- 생성자가 많아지면 유연성이 높아지고, public 메서드가 많아지면 SRP가 위반된다.
- 주 생성자를 하나두고, 부 생성자에서는 주생성자를 사용하도록 하자.
- 주 생성자가 있으면 유지보수가 편해진다. (ex 전제조건 체크 등)
### 생성자에는 코드가 없어야한다. 
> entity의 생성 전제조건 체크같은 코드는? 생성이 되지않도록 막아야하는등의 책임도 메서드호출시점으로 미뤄져야 맞는것인가?
> FastFail을 위반한다는 의견에대한 반박이 있는데 링크가 안먹는다 [See](https://www.yegor256.com/2015/05/07/ctors-must-be-code-free.html)
- 생성자는 수행하는 코드가 없어야하고, 할당만 해야한다.
- ex) 생성자에서 argument를 변환해서 넣지말고, 사용시점에 변환하도록 하자.  
    - 요청이 있을때 (메서드 호출 시점에) 파싱을 하게되면 시점을 자유룝게 조절할 수 있다.
    - 중복 파싱이 우려되면 decorator 패턴을 사용해서 캐싱하도록 하자.
- 클래스의 미래를 알 수없다. 리팩토링 할때가 되어서야 생성자 로직을 메서드로 옮기게 될 지도.
