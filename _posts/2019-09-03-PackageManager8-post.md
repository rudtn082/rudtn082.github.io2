---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager8"
description:
headline:
modified: 2019-09-03
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


## PackageManager8   


---------------------------------------

지난주에 strlcpy를 memcpy로 변경해보았는데, 속도는 빨라졌지만, 동일하게 동작하는지 알 수 없었다.  
먼저 우리가 모르는 부분에 대해서 공부를 해보기로 했다.  

### strlcpy()  

Version : android-6.0.1_r77  
소스 코드 : https://android.googlesource.com/platform/system/core/+/refs/tags/android-6.0.1_r77/libcutils/strlcpy.c  

src를 dst로 siz크기만큼 copy 한다. strncpy()와 비슷해 보였지만 다음과 같은 다른 점이 있었다.  
먼저, strcpy()는 널값까지 복사가 되지만 오버플로우 문제가 있다.  
strncpy()는 널값을 상관하지 않고 n의 길이만큼 복사하게 된다. n의 길이에 따라서 널값을 복사하지 않을 때가 있고, 성능 저하 문제가 있다.  

strlcpy()는 OpenBSD 프로젝트를 통해 유래된 비표준 C 함수로, 헤더 파일을 등록하는 것이 아니었다.  
다른 점이 있다면 strlcpy()는 주어진 크기가 0이 아닌 모든 문자열에 대해 대상 문자열을 NUL 종료하도록 보장합니다.  

그리고 보통 strncpy()를 사용하면 strlen()를 사용하여 결과의 길이를 찾는 것이 일반적이기 마련인데, strlcpy()의 리턴 값을 결괏값의 길이로 하여 최종 strlen()을 찾는 것이 더 이상 필요하지 않는다.  


다음과 같은 strlcpy()의 예시를 작성해보았다.  

![PM8_1](/images/post/PM8_1.png "PM8_1")  

아래와 같이 동작을 하고 마지막에 s - src - 1 (쓰레기값 - 기본값 - 널값)으로, 결과적으로는 src 문자열 길이를 리턴해준다.  

![PM8_2](/images/post/PM8_2.png "PM8_2")  


### memcpy()  
