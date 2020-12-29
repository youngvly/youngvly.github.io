---
title: "Custom Annotation"
date: 2019-02-17 23:43:00 -0400
categories: SpringBoot
tags: spring, annotation
---

Spring 사용시 아주 유용하게 사용되는 Annotation을 직접 만들어보자.

# Annotation 
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ParamDescriptor {
    String name() default '';

    boolean required() default false;
}
```

## @Target(ElementType.*) 
어노테이션이 붙을 위치를 정해주는것,
- Field
- Method
- Parameter
- Constructor
- Local_Variable
- Annotation_Type
- Package
- Type_parameter (type parameter declaration)
- Type_use (use of type)

## @Retention(RetentionPolicy.*)
어노테이션의 범위, 용도 
- Source : 컴파일시에 class file에 추가되지 않는다.
	소스 분석 후 특정 작업을 위한 Tool로, 컴파일시에 다양한 옵션으로 처리하거나, Proxy classes파일 생성, Bean Properties, Property Editor 등 번거로운 작업을 줄일 수 있다.
	ex) @SafeVarargs , @SuppressWarnings
- Class (default) : 컴파일러에게 class file이 기록되지만, 런타임시 가상머신에 의해 유지되지 않는다.
- RunTime : compiler에게 class file이 기록되고, AtRuntimeedp VM에 의해 유지된다. Reflection으로 접근가능하다.
	ex) @Component

## @Documented
javadoc tool에서 사용될것을 명시, default = false

## @Inherited
superClass를 상속할 수 있다 명시.

## @Repeatable
동일 어노테이션이 한번이상 용도 선언이 가능하다는것을 명시 (java 8)

참고 : https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html
