---
title: "cannot resolve import assertj, Assertions On IntelliJ"
date: 2020-12-29 21:57:00 -0400
categories: ETC
tags: intellij
---
# cannot resolve import assertj, Assertions

잘 쓰고있던 인텔리제이에서, 어느날 갑자기 test에서 assertThat import가 안되기 시작했다.

![assertJ-error](https://user-images.githubusercontent.com/19251378/103285730-46142180-4a22-11eb-9b15-8e46c154a24c.png)

근데 또 이상하게 편집기에서 빨간줄은 빡빡 그이면서, assertThat 사용중이던 테스트는 잘만 돌아갔다..!

다른 프로젝트에서는 import까지도 정상적으로 동작했었고, 특정 프로젝트에서만 발생하는 중이다.

해보았던 방법

1. Gradle refresh dependencies

2. Gradle refresh gradle project

3. intellij 캐시삭제 : File > Invalidate Cache/Restart

4. .idea 삭제 후 프로젝트 재생성

5. gradle 파일 에서 testCompile → testImpletation , compile 등등 될때까지 바꿔보기

모두 소용없었다. ㅠㅠ

이슈가 있던 프로젝트에서는 Assertions에서 우클릭 > Goto > Declaration 이 동작하지않았고,

타 프로젝트에서는 동작도 되었고 프로젝트파일 위치 focus를 했을때 

![jar-expanded](https://user-images.githubusercontent.com/19251378/103285744-50ceb680-4a22-11eb-8d38-fcb964ce01a3.png)

위 처럼 정상적으로 나왔었다.

이슈가 있던 프로젝트에서는 pom.xml만 보였고, .jar 파일이 열리지 않는 상태였다.

jar파일이 들어있는 그래들 캐시에 의심이갔고, 

Project Structure에서 jar 위치에 타고들어가

![where-is-gradle](https://user-images.githubusercontent.com/19251378/103285758-5926f180-4a22-11eb-9ada-9551c50163dd.png)

캐싱된 곳에서 3.11.1 버전 폴더를 통으로 삭제했다.

그 후 gradle refresh dependencies 하니 

![resolved](https://user-images.githubusercontent.com/19251378/103285776-6348f000-4a22-11eb-90cb-68a261ea2d0c.png)

너무나도 깔끔하게 동작...

프로젝트 삭제하고 왜 난리친거람..흑흑