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

다음으로 **early-boot, boot, property** 섹션에 해당하는 명령어를 실행한다.

다음으로 이벤트 처리 루프에 들어가기 전 감시할 이벤트 설정을 한다.  
poll()함수에서 이벤트를 기다리고 이벤트가 발생하면 poll() 함수를 빠져나와 이벤트를 처리한다.  
**① 디바이스 노드 생성 ② 프로퍼티 서비스 요청 ③ SIGCHLD 시그널 처리**  
이 세가지를 위한 파일 디스크립터를 등록하고, 아래와 같은 **이벤트 처리 루프**에서 poll() 함수의 인자로 넘겨져 이벤트를 감시한다.  

```
for(;;) {  
    drain_action_queue();  // 이벤트 처리 루프를 한번 수행 한 이후 액션 리스트, 서비스 리스트에서 실행되지 않은 명령들을 확인하고 실행.  
    restart_processes();  // 자식 프로세스가 종료됐을 때 자식 프로세스 재시작 or 종료.  

    nr = poll(ufds, fd_count, timeout);  // 등록한 파일 디스크립터에 발생한 이벤트를 기다린다. 이벤트가 발생하면 ufds에 이벤트 정보가 담김.  

     if (ufds[2].revents == POLLIN) {  //  자식 프로세스가 종료되어 SIGCHLD 시그널이 발생하면 POLLIN 이벤트 등록.  
         /* we got a SIGCHLD - reap and restart as needed */  
         read(signal_recv_fd, tmp, sizeof(tmp));  
         while (!wait_for_one_process(0));  
         continue;  
     }  

     if (ufds[0].revents == POLLIN)  
         handle_device_fd(device_fd); // 핫플러그 장치가 삽입됐을 때 디바이스 노드 파일 생성.  
     if (ufds[1].revents == POLLIN)  
         handle_property_set_fd(property_set_fd);  
     if (ufds[3].revents == POLLIN)  
         handle_keychord(keychord_fd);  
 }  
```

### init.rc 파일 분석 및 실행

init.{hardware}.rc와 init.rc가 동일하게 처리하기 때문에 init.rc에 대해서만 분석한다.  
**init 프로세스는 안드로이드를 빌드해야 생성되지만, init.rc 파일은 안드로이드 플랫폼의 소스코드에서 살펴볼 수 있다.**  
**/rootdir/init.rc**  

init.rc 파일의 내용은 크게 'on' 키워드의 액션 리스트, 'service' 키워드의 서비스 리스트로 나뉜다.  
**액션 리스트 = 시스템 환경 변수, 부팅 시 필요한 디렉터리 생성, 퍼미션 지정**  
**서비스 리스트 = 부팅시 실행하는 프로세스 기술**  

#### 액션 리스트  

액션 리스트는 **'on init'** 섹션에서 환경변수를 등록하고, 디렉터리 생성 및 마운트한다.  
안드로이드의 루트 파일 시스템 구조는 크게 **shell 유틸리티**, 라이브러리, 기본 애플리케이션을 제공하는 **system 디렉터리**, 개발자가 탑재한 사용자 애들리케이션이나 사용자 데이터를 저장하는 **data 디렉터리로** 나뉜다.  
마운트 부분에서는 /system과 /data 디렉터리를 마운트한다.  

**'on boot'** 섹션에서는 애플리케이션 종료 조건 설정, 애플리케이션 구동에 필요한 디렉터리 및 파일 퍼미션 설정 등을 한다.  
애플리케이션 종료 조건 설정부분에서 애플리케이션 별 **OOM(Out Of Memory)** 조정 값(Adjustment Value)을 지정한다.  
OOM은 커널 상에서 메모리를 모니터링하면서 메모리가 부족할 때 애플리케이션을 종료시키는 역할을 한다.  
*ADJ값이 높을수록 종료 우선순위가 높다.*  

**'on property'** 섹션에서는 프로퍼티 값이 변경될 경우 실행되는 명령이 기술돼 있다.

#### 서비스 리스트  

**'service'**섹션은 앞서 말한듯이 init 프로세스가 실행시키는 프로세스를 기술한다.
*해당 프로세스에는 부팅음을 출력하는 일회성 프로그램 또는 백그라운드의 애플리케이션이나 시스템 운용에 관여하는 데몬 프로세스가 있다.*  

'service' 섹션의 프로세스는 모드 서비스 리스트에 등록되며, init 프로세스가 실행되면서 서비스 리스트에 등록된 프로세스를 순차적으로 실행한다.  

#### init.rc 파싱 코드 분석  

parse_config_file() 함수는 인자로 전달되는 파일을 읽고(read_file()), 각 문자열을 파싱한다.(parse_config())   
*/init/parser.c 파일의 parse_config_file() 함수*  

```
int parse_config_file(const char *fn)  
{  
    char *data;  
    data = read_file(fn, 0);  // 파일을 읽고  
    if (!data) return -1;  
    parse_config(fn, data);  // 문자열 파싱  
    DUMP();  
    return 0;  
}  
```

**parse_config()** 함수는 아래 코드와 같이 인자로 전달된 파일의 끝까지 각 라인을 파싱한다.  
**next_token()** 함수로 문자열을 라인 단위로 나눈 후 **lookup_keyword()** 함수를 호출한다.  
lookup_keyword() 함수는 init.rc 파일의 각 라인에서 첫 단어에 해당하는 **keyword_list 구조체 배열**에서 번호를 반환한다.  
이후에 배열 내 flag가 SECTION인지 판단하는데, SECTION flag는 "on", "service" 키워드만 존재하기 때문에 서비스 리스트만 **parse_new_section()** 함수를 수행한다.  
parse_new_section() 함수를 수행하고 나면 액션 리스트와 서비스 리스트가 완성된다.  

```
static void parse_config(const char *fn, char *s)  
{  
    for (;;) {  
        switch (next_token(&state)) { // 문자열을 라인 단위로 나눈다.  
        case T_NEWLINE:  
            if (nargs) {  
                int kw = lookup_keyword(args[0]); // 구조체 배열에서 번호를 반환한다.  
                if (kw_is(kw, SECTION)) { // flag가 SECTION인지 확인한다.  
                    state.parse_line(&state, 0, 0); // 서비스 리스트만  
                    parse_new_section(&state, kw, nargs, args); // parse_new_section() 함수를 수행한다.  
                }  
            }  
        }  
    }  
}  
```

각 리스트는 **KEYWORD 매크로**를 사용해 생성된다.  
KEYWORD 매크로는 parse.c와 keyword.h에 정의돼 있다.  
**parse.c에 정의된 KEYWORD 매크로 = KEYWORD 리스트를 keyword_list 구조체 배열로 변환**  
**keyword.h에 정의된 KEYWORD 매크로 = KEYWORD 리스트들을 1번부터 순서대로 번호 할당**  

*parse.c에 정의된 KEYWORD 매크로*  
COMMAND 그룹은 init 프로세스가 실행하는 명령어들을 의미, 해당 명령어의 실행 함수와 매핑.  
SECTION 그룹은 액션 리스트, 서비스 리스트를 구분.  
OPTION 그룹은 명령어, 프로세스를 실행할 때 실행 조건을 부여.  
KEYWORD 리스트들은 KEYWORD 매크로를 통해 keyword_info 구조체 배열의 리스트로 변경.  

*keyword.h에 정의된 KEYWORD 매크로*  
"K_키워드" 심볼로 정의된 열거형 데이터로 변환되어 1부터 차례대로 번호가 부여된다.  

### 액션 리스트 및 서비스 리스트의 실행  

#### 액션 리스트  

액션 리스트 내의 명령어가 실행되는 과정을 먼저 살펴본다.  
action_remove_queue_head() 함수를 통해서 전역으로 선언된 액션 리스트 헤더를 얻어온다.  
액션 리스트를 command 구조체로 변환하고, 각 명령어에 매핑된 함수를 가져온다.  

```
void drain_action_queue(void)  
{  
    struct listnode *node;  
    struct command *cmd;  
    struct action *act;  
    int ret;  

    while ((act = action_remove_queue_head())) { // action_queue 연결리스트 헤더를 얻어온다.  
        INFO("processing action %p (%s)\n", act, act->name);  
        list_for_each(node, &act->commands) {  
            cmd = node_to_item(node, struct command, clist); // 리스트를 command 구초체로 변환한다.  
            ret = cmd->func(cmd->nargs, cmd->args); // 각 명령어에 매핑된 함수  
            INFO("command '%s' r=%d\n", cmd->args[0], ret);  
        }  
    }  
}  
```

#### 서비스 리스트  

'on boot' 섹선의 마지막 명령어인 class_start 명령어를 통해 service 섹션의 모든 프로세스를 실행하게 된다.

```
service_for_each_class(args[1], service_start_if_not_disabled);  
```
