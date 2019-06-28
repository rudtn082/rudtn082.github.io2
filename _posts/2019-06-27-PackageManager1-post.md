---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager"
description:
headline:
modified: 2019-06-27
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


### 주제 선정  

지난 학기 동안 '인사이드 안드로이드' 책을 통해서 안드로이드의 구조에 대해서 전반적으로 파악하는 시간을 가졌다.  
또한 6월 1일에 진행되었던, 이원영 선배님께서 멘토링을 통하여 Common Framework에 대한 내용과 개선을 어떻게 진행하면 좋을지에 대하여 다루어 주셨다.  
우리 팀은 이를 종합하여 기존에 설정하였던 전반적인 안드로이드 개선에 대한 주제를 구체화하기로 했다.  

#### 부팅시간 개선  

우리는 부팅시간 개선이라는 주제를 선정하였다.  
이를 위하여 여러 가지 방법이 있겠지만, 부팅시간에 영향을 줄만한 Package Scanning 시간을 줄이는 것을 세부 목표로 잡았다.  
Scanning 시간을 줄이기 위해서 PackageManager 쪽 코드를 공부해보고 Package Scanning 시간에 불필요한 Scanning을 하는지를 알아보려고 한다.  

##### 부팅 로그 출력  

먼저, 부팅 시에 Package Scanning에 대한 흐름을 보아야 한다.  
단말기를 재부팅시키고 로그를 확인해보는 과정을 가졌다.  

이를 확인하고자 하는 단말기는 갤럭시 A8 2016 (SM-A800S)으로, 6.0.1 버전으로 다운그레이드 하였다.  
먼저, 안드로이드 스튜디오의 Logcat기능을 통해서 로그를 확인해보았다.  
PackageManager로 태그를 걸어 PackageManager부분만 확인한 결과는 다음과 같다.  

![PM1_1](/images/post/PM1_1.png "PM1_1")  

이 방법을 사용할 때는 단말기가 꺼져있을 때 오프라인으로 되어있어, 부팅 중 로그를 출력하지 못하는 것 같다.  

다음 방법으로 adb shell을 이용하여 로그를 출력해보았다.  

![PM1_2](/images/post/PM1_2.png "PM1_2")  

안드로이드 구조를 공부했던 것 처럼 먼저 start init process 부분이 있었다.  
더 내려가서 확인해보니, 우리가 확인해야할 부분인 PackageManager 부분을 확인할 수 있었다.  

![PM1_3](/images/post/PM1_3.png "PM1_3")  

Start PackageManagerService  

![PM1_4](/images/post/PM1_4.png "PM1_4")  

End PackageManagerService  

다음 주 부터는 로그를 확인하여 불필요한 Scanning이 있는지 분석할 예정이다.  



##### adb shell 명령어  

adb shell의 명령어 같은 경우에는 안드로이드 개발자 문서에 설명되어 있다.  

https://developer.android.com/studio/command-line/adb?hl=ko#shellcommands  




### Android Package Manager and Package Installer  

원문 : http://kpbird.blogspot.com/2012/10/in-depth-android-package-manager-and.html  



#### Package Manager와 Package Installer는 무엇인가?  
PackageInstaller는 Android에 Package를 설치하는 기본 Application이다.  
PackageInstaller는 Application/Package를 관리하기 위한 사용자 인터페이스를 제공한다.  
PackageInstaller는 InstallAppProgress Activity를 호출해 사용자의 명령을 전달받는다.  

InstallAppProgress는 indalld를 통해 Package 설치를 Package Manager Service에게 요청한다.  
소스코드는 <Android Source>/packages/apps/PackageInstaller 에서 확인할 수 있다.  

installd 데몬의 가장 중요한 역할은 Linux 데몬 소켓인 /dev /socket /installed를 통해 Package Manager Service로 부터 요청을 받는 것이다. installd는 Root Permission으로 APK 설치를 위한 단계를 수행하게 된다.  
[Ref: https://github.com/android/platform_frameworks_base/blob/master/cmds/installd/commands.c]  


PackageManager는 Application 설치, 삭제, 업그레이드를 위한 API이다.  
APK파일을 설치할 때, PackageManager는 Package (APK) 파일을 분석해 확인창을 보여주고, OK 버튼을 누르면 PackageManager가 4개의 Parameter를 갖는 InstallPackage 메소드를 호출한다. (Paramter : uri, installFlags, observer, installPackageName)   PackageManager는 Package 서비스를 실행하고 Package 서비스에서 분산이 이루어진다.  
PackageInstaller 소스코드의 "PackageInstallerActivity.java" 와 "InstallAppProgress.java"를 확인해 보자.  
system_service 프로세스로 동작하는 Package Manager 서비스와 install 데몬 (installd)은 system 부팅 시점에 native process로 동작한다.  


#### Package Manager와 Package Installer의 소스코드를 어디서 찾을 수 있는가?  
< Package Manager >  
frameworks/base/services/java/com/android/server/pm/Settings.java  
frameworks/base/services/java/com/android/server/pm/PackageManagerService.java  
frameworks/base/services/java/com/android/server/pm/IPackageManager.aidl  
frameworks/base/services/java/com/android/server/pm/PackageSignatures.java  
frameworks/base/services/java/com/android/server/pm/PreferredActivity.java  
frameworks/services/java/com/android/server/PreferredComponent.java  
frameworks/core/java/android/content/IntentFilter.java  
frameworks/base/core/java/android/content/pm/PackageParser.java  
frameworks/base/services/java/com/android/server/pm/Installer.java  
frameworks/base/core/java/com/android/internal/app/IMediaContainerService.aidl  
frameworks/base/packages/DefaultContainerService/src/com/android/defcontainer/DefaultContainerService.java  

< Package Installer >  
packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageInstallerActivity.java  
packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageUtil.java  
packages/apps/PackageInstaller/src/com/android/packageinstaller/InstallAppProgress.java  
