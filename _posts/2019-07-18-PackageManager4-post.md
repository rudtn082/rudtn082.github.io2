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

1. permission, permissiongroup, permissiontree  
2. activity, activityalias, provider  
3. service, package, instrumentation  

의 메소드를 중에서 나는 3번 service, package, instrumentation 메소드 리뷰를 맡았다.  

### 코드 분석  

Version : android-6.0.1_r77  
PackageParser.java  

소스 코드 : https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.1_r77/core/java/android/content/pm/PackageParser.java  

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
