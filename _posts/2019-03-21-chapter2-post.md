---
layout: post
title: "[인사이드 안드로이드] 챕터 2 - 안드로이드 개발 환경 구축"
description:
headline:
modified: 2019-03-21
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


## Chapter2 - 안드로이드 개발 환경 구축


---------------------------------------


### 호스트 환경 구성

우분투 리눅스 상에서 안드로이드 플랫폼을 빌드하고 프레임워크를 디버깅하려면 우분투 개발활경을 구축해야한다.

*우분투 버전이 책과 달라 환경구성은 아래 블로그를 참고했습니다.*  
*http://bitly.kr/7YftX*

#### 설치과정

* Ubuntu 14.04.6 LTS를 VirtualBox를 이용하여 설치한다.  
*추후에 필요하면 가상머신이 아닌 멀티부팅을 이용해 재설치할 예정.*  

![Alt text](/images/post/install1.PNG "설치1")

![Alt text](/images/post/install2.PNG "설치2")

* 필요 패키지를 설치하고, Repo 설치 및 안드로이드 소스코드를 다운로드 한다. 책에 있는것과 마찬가지로 froyo버전을 받았다.

![Alt text](/images/post/install3.PNG "설치3")

* 안드로이드 플랫폼 빌드 확인을 위해 안드로이드 플랫폼 소스의 최상위 폴더에서 make명령어를 실행했다. 하지만 다음과 같은 에러가 발생하였다.

![Alt text](/images/post/error1.PNG "에러1")

해당 에러는 build system에서 "jar" command를 찾을 수 없어서 발생한다.  
터미널에서 *which jar*을 입력하면 아무것도 뜨지않는데, 다음과 같이 해결하였다.  
> *sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/jdk1.5.0_22/bin/jar 1 (자신의 jdk 경로)*

![Alt text](/images/post/error2.PNG "에러2")

**이후로도 빌드 오류가 일어났는데, 발생한 오류 -> 해결방법 순이다.**

![Alt text](/images/post/error3.PNG "에러3")

> frameworks/base/tools/aapt/Android.mk 파일의 31번째 줄에 -fpermissive 추가  
> *LOCAL_CFLAGS += -Wno-format-y2k -fpermissive*

![Alt text](/images/post/error4.PNG "에러4")

> frameworks/base/libs/utils/Android.mk 파일의 64번째 줄에 -fpermissive 추가  
> *LOCAL_CFLAGS += -DLIBUTILS_NATIVE=1 $(TOOL_CFLAGS) -fpermissive*

**-fpermissive 옵션은 부적합한 코드를 컴파일할 수 있도록 해주는 옵션이다.**

![Alt text](/images/post/error5.PNG "에러5")

> *cd external/srec*  
> *wget "https://github.com/CyanogenMod/android_external_srec/commit/4d7ae7b79eda47e489669fbbe1f91ec501d42fb2.diff"*  
> *patch -p1 < 4d7ae7b79eda47e489669fbbe1f91ec501d42fb2.diff*  
> *rm -f 4d7ae7b79eda47e489669fbbe1f91ec501d42fb2.diff*  

![Alt text](/images/post/error6.PNG "에러6")

> *sudo apt-get install gcc-4.4*  
> *sudo apt-get install g++-4.4*  
> 
> *Sudo rm -f /usr/bin/gcc*  
> *Sudo rm -f /usr/bin/g++*  
> *Sudo ln -s /usr/bin/gcc-4.4 /usr/bin/gcc*  
> *Sudo ln -s /usr/bin/g++-4.4 /usr/bin/g++*  
>      
> *sudo apt-get install g ++ - 4.4-multilib*  

![Alt text](/images/post/error7.PNG "에러7")

> *sudo apt-get install libswitch-perl*

**위의 오류들을 수정하고 빌드에 성공했다.**

![Alt text](/images/post/build.PNG "빌드완료")


* SDK 개발 환경 구축
책에서는 이클립스를 설치했지만, 나는 안드로이드 스튜디오를 설치했는데... '.classpath'파일이 없다... 디버깅은 실패