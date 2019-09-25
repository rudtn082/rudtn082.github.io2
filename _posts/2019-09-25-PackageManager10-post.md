---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager10"
description:
headline:
modified: 2019-09-25
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


## PackageManager10   


---------------------------------------


### 100번 측정(?)  

![PM10_1](/images/post/PM10_1.png "PM10_1")  


로그를 잘 못 썼다.

측정결과 x

고치니까 이해가 안가지만 로그를 추가하면 결과값이 달라지는 것 발견 - 해결못함

### strlcpy 수정  

코드 제대로 수정

### 문제점  

말그대로 copy기 때문에 우리가 바꾼 부분이 첫 번째 부팅과 새로 앱을 깔았을 경우에서만 사용되는 것 같음
-> 검증 해야함

이게 맞다면  원래 논문 제목과 목차를 다음과 같이 하려고 했었음

그림

앞으로 방향

1. 전체 부팅시간을 측정 해보고 개선이 된다면 그대로 부팅속도 향상쪽으로
2. 부팅속도 개선이 안된다면 메모리 블록 복사를 통한 안드로이드 개선쪽으로(?)

방향을 결정하고 PPT 제작 예정

![PM10_2](/images/post/PM10_2.png "PM10_2")  

![PM10_3](/images/post/PM10_3.png "PM10_3")  
