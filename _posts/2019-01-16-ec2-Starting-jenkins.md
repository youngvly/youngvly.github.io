---
title: "AWS EC2 에 Jenkins 시작하기"
date: 2019-01-14 23:00:00 -0400
categories: SpringBoot
---

## jenkins 설치

```shell
# sudo yum update -y
# sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# sudo yum install jenkins -y
```

## java 8 upgrade
```shell
java --version

[root@ip ~]# yum search java8
Loaded plugins: priorities, update-motd, upgrade-helper
Warning: No matches found for: java8
No matches found
[root@ip ~]# yum search jdk
Loaded plugins: priorities, update-motd, upgrade-helper
===================================================== N/S matched: jdk ======================================================
copy-jdk-configs.noarch : JDKs configuration files copier
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.6.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.6.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.7.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.7.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.7.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.7.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.8.0-openjdk-headless.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-javadoc-zip.noarch : OpenJDK API Documentation compressed in single archive
java-1.8.0-openjdk-src.x86_64 : OpenJDK Source Bundle
ldapjdk-javadoc.noarch : Javadoc for ldapjdk
ldapjdk.noarch : The Mozilla LDAP Java SDK

[root@ip ~]# yum install java-1.8.0-openjdk-devel.x86_64
...
Complete!


[root@ip ~]# /usr/sbin/alternatives --config java

There are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           /usr/lib/jvm/jre-1.7.0-openjdk.x86_64/bin/java
   2           /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/java

Enter to keep the current selection[+], or type selection number: 2

[root@ip ~]# yum remove java-1.7.0-openjdk

Complete!
[root@ip ~]# /usr/sbin/alternatives --config java

There is 1 program that provides 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/java

```

## jenkins 실행

```shell
[root@ip ~]# service jenkins start
Starting Jenkins     
```
*http://{ec2주소}:8080* 로 접속하면 jenkins를 사용할 수 있다.

사이트에 접속하면 비밀번호를 입력하라고 나오는데, 아래의 파일에서 찾을 수 있다.
```shell
[root@ip ~]# cat ~~~~/var/lib/jenkins/secrets/initialAdminPassword
```
설치가 완료되면 계정정보를 입력하고 시작할 수 있다. 

- (젠킨스 관리)Manage Jenkins -> (플러그인관리)Manage Plugins
- Available(설치가능) -> "Amazon EC2" 검색 후 체크 -> 재시작없이 다운로드
  
- (젠킨스 관리)Manage Jenkins -> (시스템 설정)Configure System -> Cloud
- Add a new Cloud -> Amazon EC2선택
- 필드 입력.