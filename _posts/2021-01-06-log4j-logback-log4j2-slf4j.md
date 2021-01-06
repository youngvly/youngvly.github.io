---
title: "Logging Framework & Slf4j"
date: 2021-01-06 19:10:00 -0400
categories: SpringBoot
tags: java, springBoot, log, logging, log4j, slf4j
---
# Apache log4j-logback-log4j2, slf4j

봐도봐도 까먹는 로깅 프레임워크

# log4j - logback - log4j2

log4j : 1999년~ 초창기 로깅 프레임워크로, 2015년 5월에 EOL되었으며, log4j2로 버전업을 권고한다.

[The Apache Software Foundation Blog](https://blogs.apache.org/foundation/entry/apache_logging_services_project_announces)

### log4j → log4j2 migration

[Log4j - Migrating from Log4j 1.x](https://logging.apache.org/log4j/2.x/manual/migration.html)

1. log4j 1.x bridge

    : log4j 1.x jar 파일 대신 Log4j 2 의  log4j-1.2-api.jar 를 넣는방법.

    단, 이 방법 사용시 요구사항이 있다. 위 홈페이지 참고

2. log4j1 → log4j2로 디펜던시 수정

    자세한 내용 위 홈페이지 참고

### 세가지 성능비교

[Log4j - Performance](https://logging.apache.org/log4j/2.x/performance.html)

대략 log4j2가 log4j1, logback 에 비해 성능이 뛰어나다는 내용.

# slf4j

: Simple Logging facade for java

로깅 프레임워크의 인터페이스로, 단독사용은 불가능하며, 로깅 프레임워크와 바인딩해서 사용할 수 있다.

코드단에선 slf4j로 사용하고, 바인딩 jar만 바꾸면 로깅 프레임워크를 바꿔쓸 수 있다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

lombok에서 사용하는 @Slf4j가 바로 이것
```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class HelloWorld {
  public static void main(String[] args) {
      log.error("error!!");
  }
}
```


### 로깅 프레임워크와 바인딩 예

출처 : [http://www.slf4j.org/manual.html](http://www.slf4j.org/manual.html)

- slf4j-log4j12-1.7.28.jar
log4j 1.2 버전과 바인딩된 slf4j
- slf4j-jdk14-1.7.28.jar
jdk 1.4 의 java.util.logging 과 바인딩된 slf4j
- slf4j-nop-1.7.28.jar
[http://www.slf4j.org/api/org/slf4j/helpers/NOPLogger.html](http://www.slf4j.org/api/org/slf4j/helpers/NOPLogger.html) 모든 로깅 무시하는 NOP 와 바인딩
- slf4j-simple-1.7.28.jar
- slf4j-jcl-1.7.28.jar
jakarta common logging
- logback-classic-1.2.3.jar
requires logback-core-1.2.3.jar