---
layout: post
title: "[인사이드 안드로이드] 챕터 4 - JNI와 NDK(2)"
description:
headline:
modified: 2019-04-04
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


## Chapter4 - JNI와 NDK(2)


---------------------------------------


### JNI 함수 이용하기  

#### JNI 함수를 활용하는 예제 프로그램의 구조  

예제 프로그램의 전체적인 구조는 다음과 같다.  
① 네이티브 메서드가 선언된 JniFuncMain 클래스  
② JniTest 객체  
③ 네이티브 메서드가 실제 구현이 포함된 jnitest.so  

#### 자바측 코드 살펴보기(JniFuncMain.java)  

JniFuncMain 클래스는 다음과 같다.  

![jni5](/images/post/jni5.png "jni5")  

JniTest 클래스는 다음과 같다.

![jni6](/images/post/jni6.png "jni6")  

#### JNI 네이티브 함수의 코드 살펴보기  

JniFuncMain.h 헤더 파일은 javah명령어를 통해 생성한다.

![jni7](/images/post/jni7.png "jni7")  

실제 함수 코드인 jnifunc.cpp는 다음과 같다.  

![jni8](/images/post/jni8.png "jni8")  

cpp파일 까지 작성을 완료한 후 이전 포스트 처럼 .so파일을 생성한다.  
여기서는 c가 아닌 cpp이기때문에 gcc가 아닌 g++로 컴파일을 한다.  

```
g++ -I/usr/lib/jvm/java5/jdk1.5.0_22/include/ -I/usr/lib/jvm/java5/jdk1.5.0_22/include/linux -shared -fPIC jnifunc.cpp -o libjnifunc.so

sudo mv libjnifunc.so /usr/lib/

java JniFuncMain
```

JniFuncMain의 결과는 다음과 같다.  

![jni8](/images/post/jni8.png "jni9")  

#### 안드로이드에서의 활용 예  
* frameworks/base/core/jni
* frameworks/base/services/jni
* frameworks/base/media/jni


### C프로그램에서 자바 클래스 실행하기  

#### 호출 API 사용 예제  