---
layout: post
title: "[인사이드 안드로이드] 챕터 4 - JNI와 NDK(3)"
description:
headline:
modified: 2019-04-10
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


## Chapter4 - JNI와 NDK(3)


---------------------------------------


### 안드로이드 NDK로 개발하기  

안드로이드 NDK(Native Development Kit)는 애플리케이션 개발자가 JNI를 활용한 작업을 쉽게 할 수 있도록 구글에서 제공하는 개발 도구다.  


#### 안드로이드 NDK환경 설정  

1. NDK 다운로드  

http://developer.android.com/intl/ko/ndk/downloads/index.html  

2. 압축 해제  

적절한 경로에 압축을 해제한다.  

```
unzip android-ndk-r19c-linux-x86_64.zip
```

3. NDK 경로설정  

```
sudo nano ~/.bashrc
```

맨 아래부분에 다음을 추가합니다.  

```
export PATH=${PATH}:(압축푼 경로)/android-ndk-r19c
```

저와같은 경우에는 다음과 같이 입력했습니다.  

```
export PATH=${PATH}:/home/kyungsoo/android-ndk-r19c
```

다음과 같이 입력해 적용완료.  

```
source /.bashrc
```

ndk-build 명령을 실행했을 때 아래와 같이 실행되면 NDK 설정이 제대로 된 것이다.  

![ndk1](/images/post/ndk1.png "ndk1")  

#### 안드로이드 NDK개발 따라하기  

프로젝트를 먼저 생성하고 Activity의 내용을 다음과 같이 수정한다.  

![ndk2](/images/post/ndk2.png "ndk2")  

javah 유틸을 실행하여 헤더 파일을 생성한다.  

![ndk3](/images/post/ndk3.png "ndk3")  

프로젝트 밑에 jni폴더를 생성하고 JNI 네이티브 함수를 구현한다.    

![ndk4](/images/post/ndk4.png "ndk4")  

![ndk5](/images/post/ndk5.png "ndk5")  

![ndk6](/images/post/ndk6.png "ndk6")  

동일하게 jni폴더에 Android.mk 파일을 작성한다.  

![ndk7](/images/post/ndk7.png "ndk7")  

프로젝트 홈 디렉터리에서 ndk-build를 실행한다.  

![ndk8](/images/post/ndk8.png "ndk8")  

이제 프로젝트를 빌드해서 실행하면 다음과 같이 1000과 42의 합인 1042가 출력된다.  

![ndk9](/images/post/ndk9.png "ndk9")  
