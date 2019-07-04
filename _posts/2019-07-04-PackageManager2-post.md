---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager"
description:
headline:
modified: 2019-07-04
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


## PackageManager


---------------------------------------


### 디바이스 선정  

지난주 adb shell과 안드로이드 스튜디오를 통한 logcat을 출력할 때는 갤럭시 A8 2016 (SM-A800S) 기기를 임시로 사용하였다.  

팀원들과 회의 결과 코드를 수정하고 빌드하는 데 구글 레퍼런스 디바이스를 사용하는 것이 좋다고 판단하였다.  
우리가 개선하고자 하는 안드로이드 버전은 6.0.1 버전으로 현재 보유하고 있는 갤럭시 넥서스는 해당 버전을 지원하지 않았다.  

우리는 넥서스 5를 테스트 및 개선을 위한 디바이스로 선정하였고, 앞으로 해당 디바이스를 통해 개선을 진행한다.  

운영체제 : Android 6.0.1_r77  


### 빌드 환경 구성  

빌드 환경 구성은 아래의 블로그를 참고했습니다.  
https://gamdekong.tistory.com/55?category=763105  

#### repo 설치  

보유하고 있는 넥서스 5에 포팅하기 위한 android-6.0.1_r77 버전의 AOSP 소스를 받아온다.  

![PM2_1](/images/post/PM2_1.png "PM2_1")  

소스를 받아온 후 빌드하기 위한 프로그램을 설치하고, mk파일을 수정한다.  

#### 빌드(make)  

```
make
```

명령어를 통해 빌드를 수행한다.  

![PM2_2](/images/post/PM2_2.png "PM2_2")  

빌드 완료 후 에뮬레이터를 실행한다.  
나와 같은 경우는 터미널이 바로 꺼지는 경우가 발생해서, 구동 시 마다 환경 설정을 다시 했다.  

```
source build/envsetup.sh
lunch aosp_arm-eng
emulator &
```

![PM2_3](/images/post/PM2_3.png "PM2_3")  


### 코드 수정 및 빌드  

우리는 PackageManager 부분의 Package Scanning 시간을 줄이는 방식을 통해 개선을 수행하고자 하기 때문에, Scanning log를 보기위해 PackageManagerService.java 코드를 수정 후 빌드해야 한다.  

https://android.googlesource.com/platform/frameworks/base/+/a029ea1/services/java/com/android/server/pm/PackageManagerService.java#189 소스를 보게 되면,  

```
private static final boolean DEBUG_PACKAGE_SCANNING = false;
```

의 DEBUG flag값이 있다. 이 값을 true로 해놓고 빌드해서 올려 package scanning에 관련된 로그를 확인하는 것이 목표이다.  

다음과 같이 수정 후 make 명령어를 통해 빌드 하였다.  

![PM2_4](/images/post/PM2_4.png "PM2_4")  


### Scanning 결과  

디바이스에 올려서 확인하기 전, 우선 에뮬레이터를 이용해서 로그값을 확인했다.  

![PM2_5](/images/post/PM2_5.png "PM2_5")  

![PM2_6](/images/post/PM2_6.png "PM2_6")  

Scanning에 관련된 로그들이 출력된 것을 확인할 수 있었다.  

하지만 PackageManagerService.java 코드를 살펴보니, Scanning에 관련된 다른 로그들도 출력되어야 할 것 같은데, 다른 로그들은 출력되지 않았다.  

#### 더 찾아봐야 할 로그들  

DEBUG flag값을 따라서 어떤 로그들이 출력되어야 하는지 확인해보았다.  

DEBUG_PACKAGE_SCANNING을 true로 변경했기 때문에, 관련 조건문에 있는 로그들은 다음과 같다.  

* Log.d(TAG, "Scanning app dir " + dir + " scanMode=" + scanMode + " flags=0x" + Integer.toHexString(flags));  
* Log.d(TAG, "Scanning package " + pkg.packageName);  
* Log.d(TAG, "Shared UserID " + pkg.mSharedUserId + " (uid=" + suid.userId + "): packages=" + suid.packages);  
* Log.v(TAG, "Want this data dir: " + dataPath);  
* Log.d(TAG, "Registered content provider: " + names[j] + ", className = " + p.info.name + ", isSyncable = " + p.info.isSyncable);  
* Log.d(TAG, "  Providers: " + r);  
* Log.d(TAG, "  Services: " + r);  
* Log.d(TAG, "  Receivers: " + r);  
* Log.d(TAG, "  Activities: " + r);  
* Log.d(TAG, "  Permission Groups: " + r);  
* Log.d(TAG, "  Permissions: " + r);  
* Log.d(TAG, "  Instrumentation: " + r);  
