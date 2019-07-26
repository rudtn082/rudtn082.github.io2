---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager4"
description:
headline:
modified: 2019-07-18
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


## PackageManager4  


---------------------------------------

### 역할 분담  

지난주 코드 분석을 통해서 우리는 PackageManagerService.java의 **scanPackageDirtyLI** 메소드, **PackageParser.java** 를 집중적으로 봐야겠다고 파악했다.  
PackageManagerService.java 에서 PackageParser.java를 부르기 때문에, PackageParser.java를 먼저 보기로 했고, 1차적으로 다음과 같이 분담하여 분석을 하기로 했다.  

1. parsePermission, parsePermissiongroup, parsePermissiontree  
2. parseActivity, parseActivityalias, parseProvider  
3. parseService, parsePackage, parseInstrumentation  

의 메소드를 중에서 나는 3번 service, package, instrumentation 메소드 리뷰를 맡았다.  

### PackageParser.java 분석  

Version : android-6.0.1_r77  
PackageParser.java  

소스 코드 : https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.1_r77/core/java/android/content/pm/PackageParser.java  

위 9개의 메소드가 아래와 같이 동일한 형태를 보였다.  

#### 1. parseService 메소드  

Line : 3779 ~ 3899  

해당 메소드는 새로운 Service 객체인 s를 생성하여 s의 info 값의 exported, permission, flags, processName을 설정하고 마지막에 s를 리턴한다.  
(인자로 받은 패키지가 Heavy-weight applications이면 메인 프로세스에서 서비스할 수 없다는 에러를 가짐)  

#### 2. parsePackage 메소드  

Line : 752 ~ 758  

해당 메소드는 인자로 받은 packageFile이 있으면(디렉토리가 존재하면) parseClusterPackage를 실행하고, 존재하지 않으면 parseMonolithicPackage를 실행한다.  

##### 2-1. parseClusterPackage 메소드  

Line : 769 ~ 814  

주어진 디렉토리를 받아 처리한다.  

해당 메소드는 Core App일 경우에만 동작하며, 패키지 이름, 버전 코드, base APK, split names(패키지 명 나눈 것)을 검사 후 Package를 리턴한다.  

##### 2-2. parseMonolithicPackage 메소드  

Line : 827 ~ 834  

2-1과 다르게 주어진 APK파일을 받아 처리한다.  

해당 메소드는 Core App일 경우에만 동작하며, parseBaseApk를 통해 얻은 Package를 리턴한다.  


#### 3. parseInstrumentation 메소드  

Line : 2339 ~ 2397  

안드로이드에서 Instrumentation는 테스트 중인 어플리케이션을 컨트롤 하고 상태를 확인할 수 있도록 제공하는 프레임워크 라고한다.  

해당 메소드는 Instrumentation인 a를 생성해서targetPackage, handleProfiling, functionalTest 값을 채워 에러가 없는경우 a를 리턴하고 에러가 있는경우 null을 리턴한다.  

### 빌드 후 플래싱 재시도  

지난번 빌드에서 제대로 빌드하여 디바이스에 플래싱 되었는지 제대로 확인할 수 없었다.  
빌드가 제대로 확인되었는지 보기 위해 PackageManagerService.java의 scanDirLI메소드의 Log를 이용하여 확인했다.  

![PM4_1](/images/post/PM4_1.png "PM4_1")  

DEBUG_PACKAGE_SCANNING 플래그 부분의 if문을 벗겨내고, TEST용 로그를 추가하여 재빌드했다.  

![PM4_2](/images/post/PM4_2.png "PM4_2")  

로그를 확인해보니, 아래와 같이 정상적으로 출력이 되었다. 빌드와 플래싱이 제대로 되는 것을 확인할 수 있었다.  

![PM4_3](/images/post/PM4_3.png "PM4_3")  

### scanPackageDirtyLI 메소드 분석 및 향후 계획  

현재 scanPackageDirtyLI 메소드를 정밀하게 분석하기 위해 다음과 같이 주석을 달아놓은 상태이다.  

![PM4_4](/images/post/PM4_4.png "PM4_4")  

PackageParser.java를 일부만 봤을 때는 특별한 점이나 개선해야 할 부분이 있는지 확인하지 못했다.  
플래싱이 제대로 되는것을 확인했기 때문에 먼저 scanPackageDirtyLI 메소드를 정밀하게 분석하고 개선할 사항이 있는지 분석할 예정이다.  
