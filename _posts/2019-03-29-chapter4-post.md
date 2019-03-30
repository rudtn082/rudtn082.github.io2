---
layout: post
title: "[인사이드 안드로이드] 챕터 4 - JNI와 NDK(1)"
description:
headline:
modified: 2019-03-29
category: Android
tags: [Android]
imagefeature: cover_1.jpg
mathjax:
chart:
comments: true
share: ture
featured: true
---

# [인사이드 안드로이드]


## Chapter4 - JNI와 NDK(1)


---------------------------------------


### 안드로이드와 JNI  

안드로이드 프레임워크는 자바와 C/C++ 기반 모듈이 계층별로 구성돼 있다. C/C++ 레이어와 자바 레이어가 서로 상호 작용하면서 동작하고 있다.  
이처럼 프레임워크에서 유기적으로 동작하게 만들려면 자바레이어(상위)와 C/C++ 레이어(하위)를 상호 연결해 주는 매개체가 필요하다.  
자바와 C/C++ 모듈 간의 인터페이스를 가능하게 해주는 것이 바로 **JNI(Java Native Interface)**다.  


JNI는 일반적으로 다음과 같은 경우에 주로 활용한다.  
* 빠른 처리 속도를 요구하는 루틴 작성(자바는 C/C++보다 느리기 때문에 C/C++로 작성하고 JNI를 통해 자바에서 호출하는 방법)  
* 하드웨어 제어(하드웨어 제어 코드를 C로 작성하면 JNI를 통해 자바에서도 하드웨어 제어 가능)  
* 기존 C/C++ 프로그램의 재사용  


**JNI에 대해 알아야 하는 이유**  
안드로이드에서는 JNI를 이용하여 자바와 C/C++의 장점을 동시에 활용하고 있다.  
안드로이드 SDK를 토대로 만든 안드로이드 애플리케이션은 달빅 가상 머신(Dalvik Virtual Machine) 위에서 동작하는 자바 기반의 프로그램이다. 때문에 C/C++로 생성한 애플리케이션에 비해 느린 실행속도 등의 한계를 가진다. 프로그램의 주요 모듈은 자바 개발자가, 성능에 민감한 모듈은 C/C++ 개발자가 작성한다.  
안드로이드를 탑재할 장치에 안드로이드 프레임워크에서 지원하지 않는 하드웨어가 설치돼 있다면 디바이스 드라이버를 C언어로 구현하고 JNI로 매핑시켜야 한다.  


### JNI의 기본 원리 이해  


#### 자바에서 C 라이브러리 함수 호출하기  


##### 자바 코드 작성  
![jni1](/images/post/jni1.png "jni1")  

JNI 네이티브 함수와 연결할 메서드를 **native 키워드**를 이용해서 선언한다.  
네이티브 메서드가 실제로 구현돼 있는 C 라이브러리를 **System.loadLibraray() 메서드**를 호출해서 로딩한다.  


##### 자바 코드 컴파일  
작성한 소스 코드를 javac(자바 컴파일러)로 컴파일한다. 컴파일 후에 .class파일이 생성되지만 자바 가상 머신이 로딩할 C라이브러리를 찾지 못하여 실행 시 아래와 같이 오류가 출력된다.  

```
Exception in thread "main" java.lang.UnsatisfiedLinkError: no hellojni in java.library.path
    at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1684)
    at java.lang.Runtime.loadLibrary0(Runtime.java:822)
    at java.lang.System.loadLibrary(System.java:993)
    at HelloJNI.<clinit>(HelloJNI.java:6)
```


##### C 헤더 파일 생성  
javah툴을 이용하여 헤더파일을 생성한다.  
생성된 .h파일의 내용은 다음과 같다.  
![jni2](/images/post/jni2.png "jni2")  


##### C/C++ 코드 구현  
헤더파일에 정의된 함수 원형을 .c파일에 복사하여 구현을 완료한다. 이 때 **매개변수의 이름을 지정**해 주어야한다.  
![jni3](/images/post/jni3.png "jni3")  


##### C 공유 라이브러리 생성  
헤더 파일과 소스 파일을 가지고 공유 라이브러리를 만든다. 윈도우에서는 .dll파일을 생성한다.  
나는 우분투에서 진행했기 때문에 아래와 같은 명령어로 .so파일을 생성한다.  

```
gcc -I/(JAVA_HOME)/include/ -I/(JAVA_HOME)/include/linux -shared -fPIC hellojni.c -o libhellojni.so
```

**(JAVA_HOME)부분에는 JDK가 설치된 경로를 입력한다.**  
나와 같은 경우에는 다음과 같이 입력했다.  
```
gcc -I/usr/lib/jvm/java5/jdk1.5.0_22/include/ -I/usr/lib/jvm/java5/jdk1.5.0_22/include/linux -shared -fPIC hellojni.c -o libhellojni.so
```

**-shared = .so파일을 생성하기 위해 붙여준다.**  
**-fPIC = 우분투가 64bit 일 때 붙여준다.**  
**.so 파일을 만들때 파일이름 앞에 lib를 붙여줘야 한다!**  


##### lib path 추가  

```
sudo nano /etc/profile
```

nano 에디터에서 맨 아랫줄에 삽입  
export LD_LIBRARY_PATH=/usr/lib:/usr/local/lib  

```
source /etc/profile
sudo mv libhellojni.so /usr/lib/
```


##### 자바 프로그램 실행  
자바 클래스를 실행하면 앞서 작업한 내용이 제대로 실행되는 것을 볼 수 있다.  
```
java HelloJNI
```
![jni4](/images/post/jni4.png "jni4")  
