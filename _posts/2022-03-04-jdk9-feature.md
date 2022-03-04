---
title: "JAVA 8 to JAVA 17 "
date: 2022-03-03 19:10:00 +0400
categories: JAVA
tags: java17
---
# JAVA 9
### 1. Modular System
1. 구조
모듈을 선언한 파일
```java
    // src/org.astro/module-info.java
    module org.astro {
        exports org.astro;
    }
```
모듈내의 구현 파일
```java
   // src/org.astro/org/astro/World.java
    package org.astro;
    public class World {
        public static String name() {
            return "world";
        }
    }
```
모듈을 선언한 파일, 위의 `astro` 모듈을 사용한다.
```java
    // cat src/com.greetings/module-info.java
    module com.greetings {
        requires org.astro;
    }
```
모듈내 구현파일, `astro` 모듈 내 메서드를 사용한다.
```java
    // cat src/com.greetings/com/greetings/Main.java
    package com.greetings;
    import org.astro.World;
    public class Main {
        public static void main(String[] args) {
            System.out.format("Greetings %s!%n", World.name());
        }
    }
```
2.  실행
```sh
$ java --module-path mods -m com.greetings/com.greetings.Main
    Greetings world!
```
3. packaging
```sh
# v1.0으로 astro module jar 생성
$ jar --create --file=mlib/org.astro@1.0.jar \
    --module-version=1.0 -C mods/org.astro .

# greeting jar 생성, mainClass 명시
$ jar --create --file=mlib/com.greetings.jar \
    --main-class=com.greetings.Main -C mods/com.greetings .

# 각각의 jar파일 생성 확인
$ ls mlib
com.greetings.jar   org.astro@1.0.jar

# -p = --module-path
$ java -p mlib -m com.greetings
    Greetings world!
```

### 2. Http Client
java.net.http -> jdk.incubator.http 로 패키지가 이동되었다.
1. Quick GET request
```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .GET()
  .build();

HttpResponse<String> response = HttpClient.newHttpClient()
  .send(request, HttpResponse.BodyHandler.asString());
```
### 3. Process API
1) pid등을 받아올 수 있는 ProcessInformation
```java
ProcessHandle self = ProcessHandle.current();
long PID = self.getPid();
ProcessHandle.Info procInfo = self.info();
 
Optional<String[]> args = procInfo.arguments();
Optional<String> cmd =  procInfo.commandLine();
Optional<Instant> startTime = procInfo.startInstant();
Optional<Duration> cpuUsage = procInfo.totalCpuDuration();
```
2) process Kill
```java
childProc = ProcessHandle.current().children();
childProc.forEach(procHandle -> {
    assertTrue("Could not kill process " + procHandle.getPid(), procHandle.destroy());
});
```
### 4. Small Language Modifications
1. try-with-resource
기존에는 `try(A)` A 부분에 새로운 variable 선언이 필수였다면,
final이거나 effectively final이면 new를 괄호 내부에서 하지않아도 된다.
```java
MyAutoCloseable mac = new MyAutoCloseable();
try (mac) {
    // do some stuff with mac
}
 
try (new MyAutoCloseable() { }.finalWrapper.finalCloseable) {
   // do some stuff with finalCloseable
} catch (Exception ex) { }

```
2. diamond operator extension
익명 inner class 인경우에도 사용가능하다.
```java
FooClass<Integer> fc = new FooClass<>(1) { // anonymous inner class
};
```
3. interface의 private 메서드
이제 가능하다!
```java
interface InterfaceWithPrivateMethods {
    
    private static String staticPrivate() {
        return "static private";
    }
    
    private String instancePrivate() {
        return "instance private";
    }
    
    default void check() {
        String result = staticPrivate();
        InterfaceWithPrivateMethods pvt = new InterfaceWithPrivateMethods() {
            // anonymous class
        };
        result = pvt.instancePrivate();
    }
}}
```
### 6. JShell CommandLine tool
python 처럼 class, main method 없이 간단한 명령어를 수행할 수 있다.
```sh
jdk-9\bin>jshell.exe
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro
jshell> "This is my long string. I want a part of it".substring(8,19);
$5 ==> "my long string"
```
### 7. JCMD Sub-Commands
- VM.class_hierarchy -s {클래스} : 클래스 계층
- VM.flags - all : vm flag를 볼 수있다.
- VM.set_vmflag : JVM 파라미터를 재시작 없이 수정할 수있다. 
아래 예시는 돌고있는 프로세스의 `java.net.Socket` 의 클래스 계층을 볼 수 있다.
```sh
# jdcd {pid} 
jdk-9\bin>jcmd 14056 VM.class_hierarchy -i -s java.net.Socket
14056:
java.lang.Object/null
|--java.net.Socket/null
|  implements java.io.Closeable/null (declared intf)
|  implements java.lang.AutoCloseable/null (inherited intf)
|  |--org.eclipse.ecf.internal.provider.filetransfer.httpclient4.CloseMonitoringSocket
|  |  implements java.lang.AutoCloseable/null (inherited intf)
|  |  implements java.io.Closeable/null (inherited intf)
|  |--javax.net.ssl.SSLSocket/null
|  |  implements java.lang.AutoCloseable/null (inherited intf)
|  |  implements java.io.Closeable/null (inherited intf)
```
### 8. Multi Resolution Image API
`java.awt.image.MultiResolutionImage` 는 여러해상도의 이미지를 하나의 인스턴스로 캡슐화하였다.
### 9. Variable Handles
`java.util.concurrent.atomic`, `sun.misc.Unsafe` 와 같은 `java.lang.invoke.`의 VarHandle, MethodHandle이 생겼다.
### 10. Publish-Subscribe Framework
`java.util.concurrent.Flow` 는 reactive stream을 제공한다. (SubmissionPublisher)
JVM에서 도는 비동기 시스템에서도 사용가능하다.
### 11. Unified JVM Logging
현재 돌고있는 프로세스의 로깅 설정 예시
```sh
jcmd {pid} VM.log output=gc_logs what=gc
```
java 실행시에됴 -Xlog로 설정가능하다.
```sh
# gc로그를 debug레벨로 gc.txt파일에 저장하라.
java -Xlog:gc=debug:file=gc.txt:none ...

```
### 12. NEW APIs
1. Immutable Sets
```java
Set<String> strKeySet = Set.of("key1", "key2", "key3");
```
2. optional to stream
```java
List<String> filteredList = listOfOptionals.stream()
  .flatMap(Optional::stream)
  .collect(Collectors.toList());
```

---
출처
- [baeldung](https://www.baeldung.com/new-java-9)
- [Module quick start](https://openjdk.java.net/projects/jigsaw/quick-start)