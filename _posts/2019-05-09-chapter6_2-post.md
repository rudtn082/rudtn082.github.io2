---
layout: post
title: "[인사이드 안드로이드] 챕터 6 - 안드로이드 서비스 개요(2)"
description:
headline:
modified: 2019-05-09
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


## Chapter6 - 안드로이드 서비스 개요(2)  


---------------------------------------


### 안드로이드 시스템 서비스  

안드로이드의 시스템 서비스는 디바이스 제어, 위치 정보 제공, 알람 설정 및 통지 메시지 표시 등과 같이 시스템의 가장 기본적인 핵심 기능들을 제공한다. 이러한 시스템 서비스는 애플리케이션 프레임워크 레이어와 라이브러리 레이어에 각각 존재한다.  

![service4](/images/post/service4.png "service4")  


#### 시스템 서비스의 분류  

##### 네이티브 시스템 서비스  

네이티브 시스템 서비스는 C++로 작성돼 있으며 라이브러리 레이어에서 동작한다.  
주요 서비스로는 ① Audio Flinger와 ② Surface Flinger 등이 있다.  

**① Audio Flinger 서비스**  

Audio Flinger 서비스는 여러 안드로이드 애플리케이션의 오디오 데이터를 믹싱해서 헤드폰이나 스피커처럼 **다양한 오디오 출력 장치로 내보내는 역할**을 한다. **안드로이드장치에서 모든 오디오 데이터는 Audio Flinger를 거쳐 출력된다.**  

![service5](/images/post/service5.png "service5")  

**② Audio Flinger 서비스**  

Surface Flinger는 다양한 애플리케이션에서 사용중인 **Surface를 조합해 프레임 버퍼 장치로 렌더링**해주는 서비스다.  

![service6](/images/post/service6.png "service6")  


##### 자바 시스템 서비스  

자바 시스템 서비스는 안드로이드 부팅 시 SystemServer라는 시스템 프로세스에 의해 일괄적으로 실행되며, ① 코어 플랫폼 서비스와 ② 하드웨어 서비스로 나뉜다.  

**① 코어 플랫폼 서비스**  

코어 플랫폼 서비스는 일반적으로 안드로이드 애플리케이션과 직접 상호작용은 하지 않지만 **안드로이드 프레임워크가 동작하는 데 필수적인 서비스**를 말한다.  

코어 플랫폼 서비스 | 기능
---|:---
`Activity Manager Service` | 모든 액티비티에 대한 라이프 사이클 및 액티비티 스택 관리
`Window Manager Service` | Surface Flinger 위에 위치하며, 기기 화면에 그릴 내용을 Surface Flinger로 전달
`Package Manager Service` | apk파일의 정보를 로딩, 시스템에 어떤 패키지가 설치되고 로딩돼 있는지에 대한 정보 제공

**② 하드웨어 서비스**  

저수준 하드웨어 제어를 위한 API를 제공하는 서비스를 말한다.  

하드웨어 서비스 | 기능
---|:---
`Alarm Manager Service` | 타이머처럼 특정 시간 후에 애플리케이션을 실행하는 등의 동작을 수행
`Connectivity Service` | 네트워크의 현재 상태에 대한 정보를 제공
`Location Service` | 단말의 현재 위치 정보를 제공
`Power Service` | 장치의 전원 관리를 제어
`Sensor Service` | 안드로이드에 설치된 각종 센서(마그네틱 센서, 가속도 센서 등)의 센싱 값을 제공
`Telephony Service` | 전화기의 상태나 전화 서비스에 대한 정보를 제공
`Wifi Service` | AP 검색이나 연결 리스트 관리 등 무선랜 연결을 제어
