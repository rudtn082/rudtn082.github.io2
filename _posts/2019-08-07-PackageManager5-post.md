---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager6"
description:
headline:
modified: 2019-08-07
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


## PackageManager6   


---------------------------------------

### scanPackageDirtyLI  

Version : android-6.0.1_r77  
PackageManagerService.java  
Line : 6482 ~ 7545  

소스 코드 : https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.1_r77/services/core/java/com/android/server/pm/PackageManagerService.java  

PackageManagerService.java의 **scanPackageDirtyLI** 메소드 중 오래걸리는 파트를 확인하기 위해 다음과 같은 방법을 통해 로그를 찍었다.  

```
long scanStart = SystemClock.uptimeMillis();

:
:

Slog.i(TAG, "Time to scan part 1: "
                  + ((SystemClock.uptimeMillis()-scanStart)/1000f)
                  + " seconds");
scanStart = SystemClock.uptimeMillis();
```


로그는 다음과 같이 출력되었으며 그 아래와 같이 평균을 내었다.  

![PM6_1](/images/post/PM6_1.png "PM6_1")  


##### 파트1 - 6482 ~ 6555 : 평균 0.0004초  
##### 파트2 - 6565 ~ 6666 : 평균 0.0000714초  
##### 파트3 - 6668 ~ 6822 : 평균 0.0000371초  
##### 파트4 - 6824 ~ 6961 : 평균 0.0000928초  
##### 파트5 - 6963 ~ 7148 : 평균 0.0315초  
##### 파트6 - 7153 ~ 7545 : 평균 0.0000714초  

라는 결과를 얻었다.  
로그를 통해서 파트5부분이 가장 오래걸리며, 해결이 시급한 부분으로 보인다.  
