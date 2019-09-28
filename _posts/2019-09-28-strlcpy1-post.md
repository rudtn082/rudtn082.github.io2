---
layout: post
title: "[안드로이드 프레임워크 개선] - strlcpy1"
description:
headline:
modified: 2019-09-28
category: Android
tags: [Android]
imagefeature: cover_2.jpg
mathjax:
chart:
comments: true
share: ture
featured: true
---

# [안드로이드 프레임워크 개선]  


## strlcpy1   


---------------------------------------

### 안드로이드 소스코드 구성    

bionic/ - 안드로이드 표준 C 라이브러리  
bootable/ - 부트로더 및 Disk Installer 등  
build/ - Makefile 관련 세팅파일들, script, map file  
development/ - 개발 시 필요한 utility & application  
external/ - 안드로이드 프레임워크가 아닌 외부에서 가져온 라이브러리와 바이너리 위치  
frameworks/ - android framework, C/C++(JNI포함)/JAVA source들, 일부 HAL source도 포함  
　　　　/base - Android Framework source  
　　　　/libs : Android base library  
　　　　/audioflinger : Android audio service & HAL source  
　　　　/surfaceflinger : Android video service & HAL  
　　　　/ui :Application Framework에서 JNI를 통해서  호출되는 Android framework의 client part, HAL (Input device의 경우)  - EventHub.cpp  
　　　　/utils : wrapping class, 압축관련 유틸리티 등  
　　　　/binder : Android Binder & Anonymous shared memory  
　　　　/cmds : binder관련인 service manager소스와 여러가지 command들  
　　　　/media : media관련 client & service library  
hardware/ - HAL source & include, 일반적으로 android에서 사용하는 hardware관련 소스들을 포함, 반드시 이 디렉토리에만 위치하는 것은 아니다(vendor 디렉토리에 존재하는 경우도 많음)  
out/ - 컴파일 된 결과물이 생성되는 디렉토리  
packages/ - android 기본 application source(JAVA)  
prebuilt/ - compiler & binaries  
system/ - android의 기본 binary 소스  
      /core/init :  android init source  
      /core/vold :  external storage 제어 모듈  


### strlcpy가 사용되는 곳 정리  

환경 : Android-6.0.1_r77  

![str_1](/images/post/str1_1.png "str1_1")  


링크 : https://docs.google.com/spreadsheets/d/1GucItCNYztFqANwKHoCE1N-0Nap_DhEjrBPiBnQGsL0/edit?usp=sharing  

안드로이드 6.0.1_r77 버전 기준 총 488개의 파일에서 1568개의 strlcpy()가 사용 중이고, 오디오나 미디어쪽 하드웨어쪽 소스에서 가장 많이 사용하는 것 같다.  
