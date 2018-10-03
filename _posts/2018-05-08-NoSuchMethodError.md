---
layout: post
title:  "java.lang.NoSuchMethodError:"
date:   2018-05-08 08:43:59
group: 개발일지
tags:   java 
---

서버에 요청시 아래와 같은 에러가 발생하여, 해결 과정을 정리했다. NoSuchMethodError이지만 해당 메서드는 소스코드 상에서는 분명히 존재하는 메서드이다. MimeUtility는 javamail 코드를 커스텀하게 수정하여 사용하고 있는 클래스로 이슈가 발생한 프로젝트의 javax.mail.internet 패키지에 별도로 클래스 파일을 관리하고 있다. 

```
java.lang.NoSuchMethodError: javax.mail.internet.MimeUtility.javaCharsetPlain(Ljava/lang/String;)Ljava/lang/String;
```

<br/> 

## 원인 
NosuchMethodError는 기본적으로 컴파일시에 사용했던 소스코드와 다른 소스코드가 런타임시 사용되는 경우 발생한다. 그래서 런타임시에만 확인할 수 있다.
Intellij에서 런타임시 사용하는 의존성 관리는 ProjectSettings → Modules → dependencies 를 보면 확인 가능하다. 

<a href="//underlinee.github.io/assets/img/20180508-1.png" data-lightbox="falcon9-large">
  <img src="//underlinee.github.io/assets/img/20180508-1.png"/>
</a>

jdk가 맨 상단에 설정되어 클래스 로딩시에 커스텀하게 로딩한 클래스보다 먼저 로딩되어 직접 정의한 클래스를 읽지 못하고 있었다. 

<a href="//underlinee.github.io/assets/img/20180508-2.png" data-lightbox="falcon9-large">
  <img src="//underlinee.github.io/assets/img/20180508-2.png"/>
</a>

이때 java dependency를 아래로 내리면 NosuchMethod 에러가 발생하지 않게 되는데, pom.xml 를 수정하면 이 설정이 다시 초기화 되면서 다시 NosuchMethod 에러가 발생하게 된다. 이전에는 해당 이슈가 발생하지 않았는데 문제가 발생한 프로젝트 패키지 구조에 변경이 있었다. 아마 관련해서 문제가 발생한 것으로 보인다. 

그리고 Intellij에서는 JDK를 모듈 dependency에서 따로 관리하면서 문제가 생기는 것으로 보이는ㄷ, 따로 JDK dependency를 관리하지 않는 remote 환경에서는 같은 문제가 발생하지 않았다. 


<br/> 


## java classloader

컴파일되어 .class 파일이 된 class는 사용하려고 할 때 classloader에 의해 메모리에 적재된다. 자바에는 기본적으로 세가지 클래스 로더가 있다. 

- Bootstrap Class Loader : rt.jar 나 java.lang.\*에 포함된 클래스 같이 JDK 내부 클래스를 로딩한다. 
- Extensions Class Loader: $JAVA_HOME/lib/ext 디렉터리에 있는 JDK 확장 라이브러리들을 로딩한다. 
- System Class Loader: 사용자가 지정한 classpath에 있는 클래스를 로딩한다. 


Bootstrap - Extensions - System 순으로 계층적으로 구성되어 있으며, 클래스가 로딩되어 있지 않다면 부모 클래스 로더가 먼저 로딩하고, 부모 클래스 로더에서 발견하지 못한 경우 다시 자식 클래스로더가 로딩하도록 설계되어 있다. 

이번 이슈의 경우 빌드시 사용하는 패키지에는 해당 클래스가 exclude 되도록 설정되어 있지만 Intellij 모듈 설정에서는 JDK를 따로 설정해서 사용하며, 우선순위가 높게 되어 있기 때문에 해당 문제가 발생했다. 


<br/> 
