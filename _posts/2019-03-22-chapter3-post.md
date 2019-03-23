---
layout: post
title: "[인사이드 안드로이드] 챕터 3 - init 프로세스(1)"
description:
headline:
modified: 2019-03-22
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


## Chapter3 - init 프로세스(1)


---------------------------------------


### init 프로세스의 실행 과정

먼저 커널소스를 받기위해 깃을 사용했다.  
```
sudo apt-get install git  
git clone https://android.googlesource.com/kernel/common.git kernel  
cd kernel  
git branch -r  
```

위의 명령어로 브랜치 정보를 확인해서, 나는 3.18버전을 체크아웃 받았다.
```
git checkout origin/android-3.18  
```

#### init_post()함수  

책에서는 init_post()함수에 대해서 설명했는데, 내가 3.18버전을 받아서 그런지 비슷한 코드의 함수명이 달랐다.  
3.18에서는 __ref형을 반환하는 kernel_init() 함수.  

![Alt text](/images/post/ch3.PNG "ch3")

책과 다른 점이 있었는데, run_init_process의 리턴값을 받아서 false일 경우에 바로 종료시키는 구문이 추가되었다.  
또한 /sbin, /etc, /bin 디렉터리에서 init파일을 못찾았을 경우 커널 패닉이 일어날 경우에 대한 메시지가 있었다.  
이 과정을 정상적으로 수행하면 init 프로세스를 실행한다.

### init 프로세스의 소스 코드 분석

main()에서 모든 프로세스의 부모인 init 프로세스는 자신이 생성한 프로세스가 종료됐을 때 발생하는 SIGCHLD 시그널을 처리할 핸들러를 등록한다.
*리눅스의 프로세스들은 정보 교환으로 메시지를 주고 받는데 이를 시그널이라 하고, 처리하기 위한 루틴을 시그널 핸들러라고 한다.*
```
sigaction(SIGCHLD, &act, 0);
```
init 프로세스는 시그널 핸들러를 등록하고 부팅에 필요한 디렉터리를 생성하고 마운트한다.

init.rc을 파싱하고 에뮬레이터 환경을 위해 QEMU 초기화를 한다. 부트 관련 서비스 초기화 후 부팅 로고를 출력한다.  
