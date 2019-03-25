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
(sigaction 구조체인 act에 값을 넣음.)  
sigaction(SIGCHLD, &act, 0);
```

init 프로세스는 시그널 핸들러를 등록하고 부팅에 필요한 디렉터리를 생성하고 마운트한다.  
이렇게 생성된 /dev, /proc, /sys와 같은 디렉터리는 init 프로세스가 동작 중에 생성하고 시스템이 종료되면 다시 사라진다.  

디렉터리가 생성된 이후에 **open_devnull_stdio()** 함수를 통해 실행 로그를 출력하기 위한 장치를 생성한다. 입출력에 관련된 파일이 모두 **__null__** 장치로 변경되어 open_devnull_stdio() 함수를 수행한 init 프로세스는 표준 입출력을 통해 메시지를 출력할 수 없다. 해당 프로세스들은 **log_init()** 함수를 통해 로그 메시지를 출력하기 위한 새로운 출력 장치를 제공한다. log_init() 함수는 **__kmsg__** 디바이스 노드 파일을 생성한다. 이는 커널의 메시지 출력 함수인 printk() 함수를 사용하게 하고 이 장치를 통해 로그 메시지를 출력한다.  

안드로이드는 init.rc와 init.{hardware}.rc 파일을 이용해 실행 파일과 환경 변수를 정의한다.   
**init.rc** 파일은 안드로이드의 공통적인 환경설정 및 프로세스  
**init.{hardware}.rc** 파일은 플랫폼에 따라 특화된 프로세스나 환경 설정 등을 정의한다.  
출력 장치를 생성한 이후 init.rc 파일을 파싱하게된다.  
파싱한 후 서비스 리스트와 액션 리스트를 연결리스트 형태로 구성한다.  

**QEMU**는 PC를 위한 오픈소스 에뮬레이터이다. init.rc을 파싱한 후 에뮬레이터 환경을 위해 QEMU 초기화를 한다.  

다음은 init.rc 파일을 분석했던 것 처럼 init.{hardware}.rc 파일을 파싱한다.  
서비스 리스트와 액션리스트를 생성하여 init.rc 파일에서 생성했던 서비스 리스트와 액션리스트에 각각 추가된다.  

init 프로세스는 **'early-init, init, early-boot, boot'** 섹션에 포함된 명령어들을 순서대로 실행한다.  
action_for_each_trigger()함수를 이용해 early-init 섹션의 명령어들을 실행 큐인 action_add_queue_tail 큐에 저장하고 drain_action_queue()를 이용해 명령어를 실행한다.  

```
action_for_each_trigger("early-init", action_add_queue_tail);  
drain_action_queue();  
```

다음으로 init 프로세스는 **device_init()** 함수를 통해 정적 디바이스 노드를 생성하고 **property_init()** 함수를 통해 프로퍼티 서비스를 초기화 한다.  
*프로퍼티 영역은 모든 프로세스에서 시스템의 설정 값을 공유하기 위해 제공된다*  

다음 단계에서 안드로이드 부팅 로고를 출력한다.  

**property_set()** 함수를 이용하여 앞서 생성한 프로퍼티 영역에 시스템 운용에 필요한 초기 값을 설정한다.  
다음은 init 프로세스의 주요 기능 중 하나인 프로퍼티 서비스를 시작한다.  

```
property_set_fd = start_property_service();  
```

init 프로세스는 자식 프로세스의 종료 처리를 위한 핸들러를 따로 정의했다.  
**socketpair()** 함수를 통해 서로 연결된 소켓 쌍 **signal_fd, signal_recv_fd**을 생성하고  
signal_fd와 signal_recv_fd의 값을 1로 설정한다.  
이벤트 처리 핸들러에서 signal_recv_fd의 값을 감시하다가 1로 설정되면서 자식 프로세스 종료 처리 핸들러를 호출한다.  

다음으로 early-boot, boot, property 섹셜에 해당하는 명령어를 싱행하고
이벤트 처리 루프에 들어가기 전 감시할 이벤트 설정을 한다.
