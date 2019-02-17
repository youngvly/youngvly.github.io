---
title: "Java - Reflection"
date: 2019-02-10 23:43:00 -0400
categories: Spring-boot-study
---

# Reflection
runTime 에 JVM에 로딩되어있는 클래스와 메소드의 정보를 읽어올 수 있다.
객체를 이용하여 클래스의 정보를 읽어올 수 있다.

## Class
- get Constructor 
- get Field
- get Method
- get Package
- get Name
...

## Method
- get ReturnType
- get ParameterTypes
- invoke (args...) : 함수수행
- get Name
...

## Field
- get Modifiers : 접근자 정보
- get Name
- get(Object)
...

## Reflection을 사용하지 않아야한다는 입장.
- 컴파일시점에 자료형을 검사하므로 exception checking 등의 이점을 포기해야한다.
- 존재하지않거나, 접근할수없는 멤버에 접근할경우 오류가 발생할것이다.
- 가독성이 떨어지며 코드가 장황하다.
- 성능이 낮다 : 리플렉션을 통한 메서드 호출의 성능은 일반 메서드 호출에 비해 훨씬 낮으며, 얼마나 낮은지는 고려해야할 조건들에 따라 달라 정확하지 않다.
+ 컴파일시점에는 알 수 없는 클래스를 이용해서 프로그램을 작성한다면 리플렉션을 사용하되, 가능하면 객체를 만들때에만 사용하고, 객체를 참조할 때에는 컴파일시에 알고있는 인터페이스나 상위 클래스를 이
용해야한다
[Effective Java#53]

## Reflection을 사용해도 된다는 입장.
- 적절히 사용한 Reflection은 오히려 성능을 향상시킬 수 있고, 많은 이득을 제공한다. 성능만을 고려한 구현이 객체지향의 설계원칙들을 역행한다면 오히려 이는 더 나쁜 결과를 낳는다.
- Reflection은 컴파일시에 타입체킹을 할 수 없어 런타임시에 잘못된 파라미터로 인해 런타임에러가 발생하기 쉽기는 하지만, 적절히 사용된 런타임 에러메세지를 이용해 충분히 디버깅이 쉬운 환경으로 만들 수 있다.
- Reflection을 사용한 코드가 일반적인 객체 생성, 메서드 호출 코드에 비교해 복잡하기는 하지만, 클래스 타입을 비교하여 객체를 생성하는 코드의 경우, if/else문을 사용하는 것보다, Reflection을 이용해 재사용가능한 컴포넌으로 만든다면 오히려 코드를 단순화시킨다.
<img src="https://redirect.viglink.com/?format=go&jsonp=vglnk_155040711028610&key=5229d68d9dbd9c51df83a3e2aa5d9234&libId=js8vugpn0102syag000DA4ehk0tmu&subId=8982&loc=https%3A%2F%2Fkmongcom.wordpress.com%2F2014%2F03%2F15%2F%25EC%259E%2590%25EB%25B0%2594-%25EB%25A6%25AC%25ED%2594%258C%25EB%25A0%2589%25EC%2585%2598%25EC%2597%2590-%25EB%258C%2580%25ED%2595%259C-%25EC%2598%25A4%25ED%2595%25B4%25EC%2599%2580-%25EC%25A7%2584%25EC%258B%25A4%2F&v=1&out=https%3A%2F%2Fkmongcom.files.wordpress.com%2F2014%2F03%2Fa5.png&ref=https%3A%2F%2Fwww.google.co.kr%2F&title=%EC%9E%90%EB%B0%94%20%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98%EC%97%90%20%EB%8C%80%ED%95%9C%20%EC%98%A4%ED%95%B4%EC%99%80%20%EC%A7%84%EC%8B%A4%20%7C%20Share%20Your%20Talents!&txt="/>
- 성능, 직접 참조경우에 비해 2~4배정도 느린것이 사실이다. 하지만, 이는 Reflection에 따른 성능저하가 아니라, 지금,성능 측정 결과, Reflection을 사용한 경우에는 성능저하가 되었다는것이다. 
JDK초기버전의 경우 Reflection의 성능이 현저히 떨어졌지만, 최근에는 많이 개선되어 Reflection을 통해 많은 이득을 취하게 해준다.


# Reflection Field를 사용한 객체의 field를 multivalueMap으로 반환해주는 유틸 

```java
public static MultiValueMap<String, String> getFieldMap(final Object o) {
		Class<?> clazz = o.getClass();
		MultiValueMap<String, String> map = new LinkedMultiValueMap<>();
		try {
			Field[] fields = clazz.getDeclaredFields();
			for (Field field : fields) {

				field.setAccessible(true);

				String fieldName = field.isAnnotationPresent(ParamDescriptor.class)
					? !field.getAnnotation(ParamDescriptor.class).name().isEmpty() ?
						field.getAnnotation(ParamDescriptor.class).name()
						: field.getName()
					: field.getName();

				if (List.class.isAssignableFrom(field.getType())) {
					//list라는것을 if 문에서 확인했음.	 request는 list<String>만을 포함함
					@SuppressWarnings("unchecked")
					List<String> array = (List<String>)field.get(o);

					map.addAll(getListMap(array, fieldName,
						field.isAnnotationPresent(ParamDescriptor.class)
							&& field.getAnnotation(ParamDescriptor.class).required())
					);
					continue;
				}

				String value = Optional.ofNullable(field.get(o)).orElse("").toString();
				if (!value.isEmpty())
					map.add(fieldName, value);
			}

		} catch (IllegalAccessException e) {
			log.error("{} | during field to param converting",e.getClass().getName());
			throw new UtilException(e.getMessage());
		}
		return map;
	}
```

객체별로 필드명을 key로, 필드값을 value로 가지는 multivalueMap을 작성하기위해,
필드명을 또하나의 MagicString으로,필드 value를 getter로 불러오거나, 클래스내부에서 작성해주는 작업이 필요했다.
 
모든 request DTO에 해당 작업을 반복해서 하는것보다, field의 이름과 value를 가져올 수 있는 Reflection을 이용해서 유틸로 두어
모든 클래스에 MultiValueMap을 작성하는 메소드를 두지않고, 유틸 메서드를 재활용함으로써 코드 효율이 높아졌다.




### Field.setAccessible(true)

reflected object가 사용될때 발생하는 Java language access control Check를 suppress한다.
- private Field를 다른 클래스에서 read, write 가능하게 한다. 
- securityMannager가 private member의 접근을 허용하지 않을경우에는 메소드실행이 실패할것.
- suppress 하지 않으면, IllegalAccessException이 발생한다. 

- Java serialization도 reflection의 set Accessible을 사용한다.
