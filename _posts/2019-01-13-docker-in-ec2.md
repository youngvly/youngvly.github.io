---
title: "Docker in EC2"
date: 2019-01-13 18:00:00 -0400
categories: SpringBoot
---

# AWS ec2에 Docker 설치부터 배포하기

1. EC2 인스턴스 생성
2. Xshell에 ssh port 22 로 연결
3. user 생성하기 
```shell
$ adduser root
```
4. EC2에 Docker 설치하기
```shell
$ sudo yum update -y
$ sudo yum install -y docker
Loaded plugins: priorities, update-motd, upgrade-helper
amzn-main                                                                      | 2.1 kB  00:00:00     
amzn-updates                                                                   | 2.5 kB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package docker.x86_64 0:18.06.1ce-5.22.amzn1 will be installed
--> Processing Dependency: xfsprogs for package: docker-18.06.1ce-5.22.amzn1.x86_64
--> Processing Dependency: pigz for package: docker-18.06.1ce-5.22.amzn1.x86_64
--> Processing Dependency: libseccomp.so.2()(64bit) for package: docker-18.06.1ce-5.22.amzn1.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-18.06.1ce-5.22.amzn1.x86_64
--> Running transaction check
---> Package libseccomp.x86_64 0:2.3.1-2.4.amzn1 will be installed
---> Package libtool-ltdl.x86_64 0:2.4.2-20.4.8.5.32.amzn1 will be installed
---> Package pigz.x86_64 0:2.3.3-1.6.amzn1 will be installed
---> Package xfsprogs.x86_64 0:4.5.0-9.21.amzn1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================
 Package               Arch            Version                            Repository             Size
======================================================================================================
Installing:
 docker                x86_64          18.06.1ce-5.22.amzn1               amzn-updates           45 M
Installing for dependencies:
 libseccomp            x86_64          2.3.1-2.4.amzn1                    amzn-main              79 k
 libtool-ltdl          x86_64          2.4.2-20.4.8.5.32.amzn1            amzn-main              51 k
 pigz                  x86_64          2.3.3-1.6.amzn1                    amzn-main              71 k
 xfsprogs              x86_64          4.5.0-9.21.amzn1                   amzn-main             1.7 M

Transaction Summary
======================================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 47 M
Installed size: 154 M
Downloading packages:
(1/5): libtool-ltdl-2.4.2-20.4.8.5.32.amzn1.x86_64.rpm                         |  51 kB  00:00:00     
(2/5): libseccomp-2.3.1-2.4.amzn1.x86_64.rpm                                   |  79 kB  00:00:00     
(3/5): pigz-2.3.3-1.6.amzn1.x86_64.rpm                                         |  71 kB  00:00:00     
(4/5): xfsprogs-4.5.0-9.21.amzn1.x86_64.rpm                                    | 1.7 MB  00:00:00     
(5/5): docker-18.06.1ce-5.22.amzn1.x86_64.rpm                                  |  45 MB  00:00:05     
------------------------------------------------------------------------------------------------------
Total                                                                 8.0 MB/s |  47 MB  00:00:05     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libseccomp-2.3.1-2.4.amzn1.x86_64                                                  1/5 
  Installing : libtool-ltdl-2.4.2-20.4.8.5.32.amzn1.x86_64                                        2/5 
  Installing : pigz-2.3.3-1.6.amzn1.x86_64                                                        3/5 
  Installing : xfsprogs-4.5.0-9.21.amzn1.x86_64                                                   4/5 
  Installing : docker-18.06.1ce-5.22.amzn1.x86_64                                                 5/5 
  Verifying  : xfsprogs-4.5.0-9.21.amzn1.x86_64                                                   1/5 
  Verifying  : pigz-2.3.3-1.6.amzn1.x86_64                                                        2/5 
  Verifying  : libtool-ltdl-2.4.2-20.4.8.5.32.amzn1.x86_64                                        3/5 
  Verifying  : docker-18.06.1ce-5.22.amzn1.x86_64                                                 4/5 
  Verifying  : libseccomp-2.3.1-2.4.amzn1.x86_64                                                  5/5 

Installed:
  docker.x86_64 0:18.06.1ce-5.22.amzn1                                                                

Dependency Installed:
  libseccomp.x86_64 0:2.3.1-2.4.amzn1          libtool-ltdl.x86_64 0:2.4.2-20.4.8.5.32.amzn1         
  pigz.x86_64 0:2.3.3-1.6.amzn1                xfsprogs.x86_64 0:4.5.0-9.21.amzn1                    

Complete!

```

5. Docker 실행
```shell
$ sudo service docker start
Starting cgconfig service:                                 [  OK  ]
Starting docker:	.                                  [  OK  ]


$ sudo usermod -a -G docker $USER
```
$USER = root(혹은 ec2-user)
sudo 없이 사용하기 위해 docker그룹에 sudo 를 추가

6. Docker 에 Tomcat 설치
```shell
$ docker pull tomcat:8      #톰캣 설치
$ docker images             #이미지 확인
$ docker run -d -i -t -p 8080:8080 tomcat:8     #실행
$ docker ps                 #실행 확인
$ docker restart $ID
$ docker rmi $Image_ID      #이미지 삭제
```
tomcat 실행시
-d : demon으로 실행한다
-p : 8081:8080 -> 이미지를 8080으로 실행하지만, 호스트에서 접근시 8081로 접근한다
-i : 표준 입력을 활성화하여 컨테이너와 연결되어있지 않더라도 표준 입력을 활성화한다.

## Docker File
Each instruction creates one layer:

- *FROM* creates a layer from the ubuntu:15.04 Docker image.
- *COPY* adds files from your Docker client’s current directory.
- *RUN* builds your application with make.
- *CMD* specifies what command to run within the container.

```shell
$ vi DockerFile

FROM tomcat:8
MAINTAINER dy1dy2@gmail.com
COPY demo-0.0.1-SNAPSHOT.war /usr/local/tomcat/lib/
EXPOSE 8080
```

7. Deploy War file
   - Dependency 추가 
    + 컨테이너가 탑재된 단독형 앱을 확장 간으한 형태로 유지할 수 있다.
    + WEB-INF/lib-provided 에 톰캣 라이브러리가 들어간다.
    + 톰캣 라이브러리가 WEB-INF/lib 에 들어가면 JAR파일이 충돌하면서 컨테이너가 시동 실패한다.
```xml
          <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```
  - packaging 추가
```xml
    <packaging>war</packaging>
```
  - Main App 수정
      + WAR파일에서 SpringApplication을 실행하는 WebApplicationInitializer 인터페이스를 스프링 부터에 맞게 구현한 클래스, 애플리케이션 컨텍스트로부터 서블릿, 필터, ServletContextInitializer 빈을 서블릿 컨테이너에 바인딩한다. 
```java
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(DemoApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
  - Packaging
```shell
   mvn clean package -DskipTests=true
#    mvn clean install spring-boot:repackage -DskipTests=true
```
   + install 을 하는 이유
   오류 발생
```shell
   :repackage failed: Source file must be provided -> [Help 1]
```
    spring-boot-maven-plugin 을 .m2 folder로 다운로드 하는중 이슈가 있는듯. m2\repository 에 있는 것을 삭제하고 재설치하는 작업이 필요.
- war 파일로 변경하고싶은 경우:  https://www.reimaginer.me/entry/Spring-Boot-jar-to-war
  
8. FTP 사용
 vsftpd를 설치한다(Very Secured FTPD) : 우분투에서 기본적으로 제공되는 ftp 서버
```shell
$ sudo yum install vsftpd
```

- fileZila 다운로드 후 SFTP 로 연결. (22번포트 사용: SSH File Transfer Protocol 이기때문)
- 해당 root 폴더에 war 파일 복사
- 해당 root 폴더에 Dockerfile 같이 있어야함.

9. docker build
    
```shell
[root@ip ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              8                   1a51cb5e3006        2 weeks ago         462MB
[root@ip ~]# docker build -t tomcat_test .
Sending build context to Docker daemon  20.06MB
Step 1/4 : FROM tomcat:8
 ---> 1a51cb5e3006
Step 2/4 : MAINTAINER dy1dy2@gmail.com
 ---> Running in 86d231eec71d
Removing intermediate container 86d231eec71d
 ---> fb4c81959c6c
Step 3/4 : COPY demo-0.0.1-SNAPSHOT.war /usr/local/tomcat/lib/
 ---> 406e47b1ac22
Step 4/4 : EXPOSE 8080
 ---> Running in 49280248ae76
Removing intermediate container 49280248ae76
 ---> b372aa986c86
Successfully built b372aa986c86
Successfully tagged tomcat_test:latest
[root@ip ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat_test         latest              b372aa986c86        4 seconds ago       483MB
tomcat              8                   1a51cb5e3006        2 weeks ago         462MB

```
마지막 . 은 dockerfile이 있는 장소인듯?

1.  Docker Run
```shell
[root@ip ~]# docker run -d -i -t --name="tomcat_test_container" -p 8080:8080 -v demo-0.0.1-SNAPSHOT.war:/usr/local/tomcat/webapps/demo-0.0.1-SNAPSHOT.war tomcat_test
c716f7e9fdf11a1c00c21ba1dea568f5fdb1b021372f048cc8746f1807b7bab4
```
-v ~파일을 /usr/local/tomcat/webaps/~ 로 copy하겠다. 
-p : 8081:8080 -> 이미지를 8080으로 실행하지만, 호스트에서 접근시 8081로 접근한다
```shell
[root@ip ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
c716f7e9fdf1        tomcat_test         "catalina.sh run"   55 seconds ago      Up 53 seconds       0.0.0.0:8080->8080/tcp   tomcat_test_container
d55adc6a242c        tomcat:8            "catalina.sh run"   2 hours ago         Up 2 hours          0.0.0.0:8081->8080/tcp   tomcat8-board

[root@ip ~]# docker exec -i -t tomcat_test_container /bin/bash
root@c716f7e9fdf1:/usr/local/tomcat# ls -al webapps/
total 36
drwxr-xr-x  1 root root  4096 Jan 13 14:30 .
drwxr-sr-x  1 root staff 4096 Dec 29 11:48 ..
drwxr-xr-x  3 root root  4096 Dec 29 11:48 ROOT
drwxr-xr-x  2 root root  4096 Jan 13 14:29 demo-0.0.1-SNAPSHOT.war
drwxr-xr-x 14 root root  4096 Dec 29 11:48 docs
drwxr-xr-x  6 root root  4096 Dec 29 11:48 examples
drwxr-xr-x  5 root root  4096 Dec 29 11:48 host-manager
drwxr-xr-x  5 root root  4096 Dec 29 11:48 manager
root@c716f7e9fdf1:/usr/local/tomcat# exit
exit
```
webapps에 파일이 들어가있는지 확인 가능. 

```shell
docker stop tomcat_test_container
docker rm tomcat_test_container
```


[1] Docker : http://tech.cloudz-labs.io/posts/docker/docker-start/
[2] 도커파일 참고해볼 사이트 : https://jistol.github.io/docker/2017/09/19/docker-compose-tomcat-clustering/