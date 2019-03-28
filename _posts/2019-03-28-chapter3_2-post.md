---
layout: post
title: "[인사이드 안드로이드] 챕터 3 - init 프로세스(2)"
description:
headline:
modified: 2019-03-28
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


## Chapter3 - init 프로세스(2)


---------------------------------------


### 디바이스 노드 파일 생성  

안드로이드에서 애플리케이션이 하드웨어에 접근할 때 **디바이스 드라이버**를 통해서 접근한다.  
애플리케이션은 디바이스 드라이버에 접근하기 위해 **디바이스 노드**를 사용한다.  
리눅스에서는 'mknod'유틸리티를 지원하지만 **안드로이드에서는 보안문제로 제공하지 않는다.**  

#### 정적 디바이스 노드 생성  

리눅스에서는 디바이스 노드파일을 "/dev" 디렉터리에 정의하고, 이 노드파일을 통해 디바이스 드라이버에 접근한다.  
하지만 안드로이드는 존재하지 않고, **init 프로세스가 두 가지 방법으로 디바이스 노드 파일을 생성**한다.

① 미리 정의된 디바이스 정보를 바탕으로 init 프로세스가 실행될 때 일괄적으로 디바이스 노드 파일 생성 - 콜드 플러그(Cold Plug)  
② 시스템 동작 중 USB와 같은 장치가 삽입될 때 이벤트 처리로 디바이스 노드 파일을 동적으로 생성 - 핫 플러그(Hot Plug)  

##### 콜드 플러그 방식  

콜드 플러그는 이미 삽입된 장치에 대한 처리를 담당하는 메커니즘.  
리눅스에서는 부팅 후에 udev 데몬이 실행되면서 "/sys" 디렉터리에서 미리 등록된 디바이스 정보를 읽고, 디바이드에 대해 uevent를 발생시켜 디바이스 노드 파일을 생성하는 방식이다.  
*udev = "/dev" 디렉터리에 자동으로 디바이스 노드 파일을 생성하는 역할*  
*uevent = 커널에서 사용자 공간의 프로세스로 메시지를 전달하기 위한 신호 체계*  

이를 안드로이드에서는 udev 데몬의 역할을 init 프로세스가 수행한다.  

책에서는 콜드 플러그 방식으로 디바이스 노드를 생성하는 예로 7장의 '안드로이드 바인더 IPC'를 통해 설명한다.  
먼저 바인더 드라이버는 가상의 장치이며 프로세스간의 RPC(Remote Procedure Call)를 제공하는 데 쓰인다.  
바인더를 이용하려는 응용프로그램은 "/dev/binder"라는 디바이스 노드를 사용해서 바인더 디바이스에 접근해야 한다.  

**커널 부팅 시**  
커널 부팅 시 바인더 드라이버는 misc_register() 함수를 통해서 디바이스 노드 파일을 생성하는 데 필요한 정보를 "/sys" 파일 시스템에 저장한다.  
이제 init 프로세스를 통해 디바이스 노드 파일을 생성해야 하는데, 아직 커널 부팅 단계이므로 uevent를 발생시킬 수 없다.  

**커널 부팅 완료**  
init 프로세스가 실행되면 위의 바인더 드라이버처럼 디바이스 노드 파일을 생성하지 못한 드라이버에 대해 콜드플러그 처리를 한다.  
콜드 플러그 처리될 드라이버들은 아래처럼 devices.c 파일에 미리 정의되어있다.  

```
static struct perms_ devperms[] = {  
    { "/dev/null",          0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/zero",          0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/full",          0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/ptmx",          0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/tty",           0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/random",        0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/urandom",       0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/ashmem",        0666,   AID_ROOT,       AID_ROOT,       0 },  
    { "/dev/binder",        0666,   AID_ROOT,       AID_ROOT,       0 },  
    :  
    :  
```

devperms 구조체를 참고하여 "/dev" 디렉터리에 디바이스 노드 파일들을 생성한다.  
구조체는 파일 이름, 접근권한, 사용자 ID, 그룹 ID를 나타낸다.  

**콜드 플러그 처리 절차**  
device_init() 함수는 uevent를 수신하기 위한 소켓을 생성하고, coldboot()를 통해 내부적으로 do_coldboot() 함수를 호출하여 "/sys" 디렉터리에 등록된 드라이버에 대해 콜드플러그 처리를 한다.
**devices.c 파일에 콜드 플러그될 드라이버를 정의해뒀는데 왜 다시 /sys 디렉터리를 통해서 콜드플러그 처리를 할까??**

```
int device_init(void)  
{  
    fd = open_uevent_socket(); // 소켓 생성  
    t0 = get_usecs(); // 시간측정  
    coldboot(fd, "/sys/class");  
    coldboot(fd, "/sys/block");  
    coldboot(fd, "/sys/devices");  
    t1 = get_usecs(); // 시간측정  
    log_event_print("coldboot %ld uS\n", ((long) (t1 - t0))); // 시간 로그 출력  
}  
```

do_coldboot() 함수에서는 디렉터리 경로를 받아 해당 경로를 이용해 저장된 uevent 파일을 찾은 후, add메시지를 써넣어 uevent를 발생시킨다.  
**파일에 add를 넣는다고 uevent가 바로 발생되는가??**  

```
static void do_coldboot(int event_fd, DIR *d)  
{  
    fd = openat(dfd, "uevent", O_WRONLY); // uevent 파일을 찾아서  
    if(fd >= 0) {  
        write(fd, "add\n", 4); // "add" 추가로 uevent 발생  
        close(fd);  
        handle_device_fd(event_fd); // uevent 수신하여 처리  
    }  
}  
```

그리고 handle_device_fd() 함수에서 uevent를 수신하여 실제 노드 파일을 생성한다.  
/dev 디렉터리 아래에 하위 디렉터리를 생성한다.  

```
static void handle_device_event(struct uevent *uevent) // uevent 구조체를 받아서  
{  
  if(!strncmp(uevent->subsystem, "block", 5)) {  
          block = 1;  
          base = "/dev/block/";  
          mkdir(base, 0755); // 하위 디렉터리를 생성.  
      }  
}  
```

하위 디렉터리를 모두 생성하면 make_device() 함수를 통해 디바이스 노드 파일을 생성한다.  

```
static void make_device(const char *path, int block, int major, int minor)  
{  
    mode_t mode;  
    dev_t dev;  
    mknod(path, mode, dev); // 디바이스 노드 파일 생성  
    chown(path, uid, -1);  
}  
```

#### 동적 디바이스 감지

init 프로세스는 시스템 동작 중 추가되는 장치의 디바이스 노드 파일 생성을 위해 핫플러그 처리를 지원한다.  

##### 핫 플러그 방식  

다음과 같이 init 프로세스의 이벤트 처리 루프에서 처리된다.  

```
int main(int argc, char **argv)
{  
  for(;;) {  
        nr = poll(ufds, fd_count, timeout); // poll 함수를 통해서 uevent를 감지  
        :  
        :  
        if (ufds[0].revents == POLLIN)  
            handle_device_fd(device_fd); // 콜드 플러그와 같이 handle_device_fd() 함수를 호출  
  }  
}  
```


### 프로세스 종료와 재시작  
