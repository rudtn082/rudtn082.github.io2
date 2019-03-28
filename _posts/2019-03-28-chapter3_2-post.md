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
하지만 안드로이드의 루트 파일 시스템은 "/dev" 디렉터리가 존재하지 않고, **init 프로세스가 두 가지 방법으로 디바이스 노드 파일을 생성**한다.

① 미리 정의된 디바이스 정보를 바탕으로 init 프로세스가 실행될 때 일괄적으로 디바이스 노드 파일 생성 - **콜드 플러그(Cold Plug)**  
② 시스템 동작 중 USB와 같은 장치가 삽입될 때 이벤트 처리로 디바이스 노드 파일을 동적으로 생성 - **핫 플러그(Hot Plug)**  

##### 콜드 플러그 방식  

콜드 플러그는 이미 삽입된 장치에 대한 처리를 담당하는 메커니즘.  
리눅스에서는 부팅 후에 udev 데몬이 실행되면서 "/sys" 디렉터리에서 미리 등록된 디바이스 정보를 읽고, 디바이드에 대해 uevent를 발생시켜 디바이스 노드 파일을 생성하는 방식이다.  
*udev = "/dev" 디렉터리에 자동으로 디바이스 노드 파일을 생성하는 역할*  
*uevent = 커널에서 사용자 공간의 프로세스로 메시지를 전달하기 위한 신호 체계*  

이를 안드로이드에서는 udev 데몬의 역할을 **init 프로세스가 수행**한다.  

책에서는 콜드 플러그 방식으로 디바이스 노드를 생성하는 예로 7장의 '안드로이드 바인더 IPC'를 통해 설명한다.  
먼저 바인더 드라이버는 가상의 장치이며 프로세스간의 RPC(Remote Procedure Call)를 제공하는 데 쓰인다.  
바인더를 이용하려는 응용프로그램은 "/dev/binder"라는 디바이스 노드를 사용해서 바인더 디바이스에 접근해야 한다.  

**커널 부팅 시**  
커널 부팅 시 바인더 드라이버는 misc_register() 함수를 통해서 디바이스 노드 파일을 생성하는 데 필요한 정보를 "/sys" 파일 시스템에 저장한다.  
이제 init 프로세스를 통해 디바이스 노드 파일을 생성해야 하는데, **아직 커널 부팅 단계이므로 uevent를 발생시킬 수 없다.**  

**커널 부팅 완료**  
init 프로세스가 실행되면 위의 바인더 드라이버처럼 디바이스 노드 파일을 생성하지 못한 드라이버에 대해 콜드플러그 처리를 한다.  
콜드 플러그 처리될 드라이버들은 아래처럼 **devices.c 파일에 미리 정의**되어있다.  

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

devperms 구조체를 참고하여 **"/dev" 디렉터리에 디바이스 노드 파일들을 생성**한다.  
구조체는 파일 이름, 접근권한, 사용자 ID, 그룹 ID를 나타낸다.  

**콜드 플러그 처리 절차**  
device_init() 함수는 uevent를 수신하기 위한 소켓을 생성하고, coldboot()를 통해 내부적으로 do_coldboot() 함수를 호출하여 "/sys" 디렉터리에 등록된 드라이버에 대해 콜드플러그 처리를 한다.  

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
*파일에 add를 넣는다고 uevent가 바로 발생되는가??*  

```
static void do_coldboot(int event_fd, DIR *d)  
{  
    fd = openat(dfd, "uevent", O_WRONLY); // uevent 파일을 찾아서  
    if(fd >= 0) {  
        write(fd, "add\n", 4); // "add" 메시지 추가로 uevent 발생  
        close(fd);  
        handle_device_fd(event_fd); // uevent 수신하여 처리  
    }  
}  
```

그리고 handle_device_fd() 함수에서 uevent를 수신하여 메시지를 수신해서 uevent구조체에 할당한다.  
uevent 구조체를 완성하면 handle_device_event() 함수를 호출하여 실제 노드 파일을 생성한다.  
**/dev 디렉터리 아래에 하위 디렉터리를 생성**한다.  

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

하위 디렉터리를 모두 생성하면 **make_device() 함수를 통해 디바이스 노드 파일을 생성**한다.  

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

다양한 프로세스들이 init 프로세스에 의해 실행되고, 종료되면 시스템 동작에 영향을 미치는 것들이 존재한다.  
따라서 init 프로세스가 실행하는 프로세스는 일부를 제외하고 **대부분은 재시작된다.**  
프로세스 옵션에 **onshot이 정의돼 있다면 재시작하지 않는다.**  

#### 프로세스 재시작 코드 분석  

자식 프로세스가 종료되면 다음과 같이 SIGCHLD 시그널에 대한 핸들러를 수행한다.  

```
static void sigchld_handler(int s) // SIGCHLD 시그널 번호를 받는다.  
{  
    write(signal_fd, &s, 1); // 소켓 디스크립터에 기록한다.  
}  
```

signal_fd는 이전에 소켓쌍으로 생성됐기 떄문에 수신측 소켓 디스크립터인 **signal_recv_fd**로 시그널 번호를 전송한다.  
시그널 번호를 받은 signal_recv_fd는 main에서 이미 POLL로 등록돼 있기 때문에 **wait_for_one_process() 함수를 실행**하게 된다.  
*poll()함수는 이벤트 처리 루프에서 이벤트를 감시했었음.*  

```
int main(int argc, char **argv)  
{  
  for(;;) {  
    nr = poll(ufds, fd_count, timeout); // poll() 함수는 이벤트를 감시하다가 빠져나옴.  
    if (nr <= 0)  
        continue;  

    if (ufds[2].revents == POLLIN) {  
        /* we got a SIGCHLD - reap and restart as needed */  
        read(signal_recv_fd, tmp, sizeof(tmp));  
        while (!wait_for_one_process(0)); // wait_for_one_process() 함수를 수행.  
        continue;  
    }  
  }  
}  
```

다음은 **wait_for_one_process() 함수**를 보여준다.  

```
static int wait_for_one_process(int block)  
{  
    :  
    :  
    while ( (pid = waitpid(-1, &status, block ? 0 : WNOHANG)) == -1 && errno == EINTR ); // 프로세스가 종료되면 할당된 자원을 회수.  

    svc = service_find_by_pid(pid); // 종료된 프로세스에 해당하는 서비스 항목을 가져옴.  

    if (!(svc->flags & SVC_ONESHOT)) { // 가져온 서비스 항목의 옵션에 SVC_ONESHOT이 설정 되어있는지 체크.  
        kill(-pid, SIGKILL); // SVC_ONESHOT이 설정 되어있으면 종료  
        NOTICE("process '%s' killing any children in process group\n", svc->name);  
    }  

    /* remove any sockets we may have created */  
    for (si = svc->sockets; si; si = si->next) { // 프로세스가 가진 소켓 디스크립터를 모두 삭제.  
        char tmp[128];  
        snprintf(tmp, sizeof(tmp), ANDROID_SOCKET_DIR"/%s", si->name);  
        unlink(tmp);  
    }  

    svc->pid = 0;  
    svc->flags &= (~SVC_RUNNING); // 프로세스가 구동중임을 나타내는 pid 값, SVC_RUNNING을 제거.  

        /* oneshot processes go into the disabled state on exit */  
    if (svc->flags & SVC_ONESHOT) { // SVC_ONESHOT 옵션이 설정된 프로세스의 플래그를  
        svc->flags |= SVC_DISABLED; // disabled 설정하고  
    }  

        /* disabled processes do not get restarted automatically */  
    if (svc->flags & SVC_DISABLED) { // disabled 설정된 프로세스는  
        notify_service_state(svc->name, "stopped");  
        return 0; // 함수를 빠져나가서 재시작되지 않는다.  
    }  

    /* Execute all onrestart commands for this service. */  
    list_for_each(node, &svc->onrestart.commands) { // 재시작할 프로세스가 onrestart 옵션을 가지는지 체크.  
        cmd = node_to_item(node, struct command, clist);  
        cmd->func(cmd->nargs, cmd->args);  
    }  

    svc->flags |= SVC_RESTARTING; // 프로세스의 플래그에 SVC_RESTARTING를 추가한다.  
}  
```

wait_for_one_process() 함수의 실행이 완료되면 **restart_processes() 함수를 실행**한다.  

```
static void restart_processes()  
{  
    process_needs_restart = 0;  
    service_for_each_flags(SVC_RESTARTING, restart_service_if_needed); // SVC_RESTARTING 플래그를 가진 프로세스를 실행한다.  
}  
```

따라서 자식프로세스가 종료되어 SIGCHLD 시그널을 발생시키더라도 이 함수를 통해 재시작하게 된다.  



### 프로퍼티 서비스  

init 프로세스의 이벤트 처리 루프에서는 프로퍼티의 변경 요청 이벤트도 있다.  
* 프로퍼티는 안드로이드 시스템이 동작하는 데 필요한 **각종 설정 값을 동작 중인 모든 프로세스에서 공유하기 위해 프로퍼티라는 저장 공간을 사용**한다.  
* 프로퍼티는 키(key)와 값(value)로 구성되며, '키 = 값' 형태로 사용된다.  
* 안드로이드에서는 이 값을 변경할 때는 권한을 확인하는 과정이 있다.  
* 모든 동작중인 프로세스는 프로퍼티의 값을 조회할 수 있다.  
* 프로퍼티 값을 **변경하는 것은 init 프로세스만이 가능**, 다른 프로세스는 변경 요청.  
* 프로퍼티의 값이 변경되면 init.rc에 정의된 특정 조건을 만족하는 경우 조건에 해당하는 동작 실행, **이를 트리거(trigger)**라 한다.  

#### 프로퍼티 초기화  

init 프로세스의 main() 함수 초기에 **property_init() 함수**를 통해서 프로퍼티 영역을 초기화한다.  
property_init() 함수는 프로퍼티의 값을 저장하기 위한 공유 메모리를 생성하는데, 이를 위해 ashmem(Android Shared Memory)을 사용한다.  
*프로퍼티 값을 저장하거나 조회할때는 get(), set() 함수를 이용한다.*  

```
void property_init(void)  
{  
    init_property_area(); // 공유 메모리 영역 구성.  
    load_properties_from_file(PROP_PATH_RAMDISK_DEFAULT); // 파일로 부터 초기값을 읽어 프로퍼티 값을 설정.  
}  
```

init 프로세스는 start_property_service() 함수를 호출하여 프로퍼티 서비스를 시작한다.  

```
property_set_fd = start_property_service(); // 프로퍼티 서비스 시작.  
```

```
int start_property_service(void)  
{  
    int fd;  
    load_properties_from_file(PROP_PATH_SYSTEM_BUILD); // 프로퍼티의 기본값을 읽어 프로퍼티 값을 설정.  
    load_properties_from_file(PROP_PATH_SYSTEM_DEFAULT);  
    load_properties_from_file(PROP_PATH_LOCAL_OVERRIDE);  

    /* Read persistent properties after all default values have been loaded. */  
    load_persistent_properties(); // /data/property 디렉터리에 저장돼 있는 프로퍼티 값을 읽는다. (동작 중에 다른 프로세스에 의해 생성된 프로퍼티 값이나 변경된 값들)  
    fd = create_socket(PROP_SERVICE_NAME, SOCK_STREAM, 0666, 0, 0); // property_service라는 이름의 도메인 소켓 생성.  
}  
```

#### 프로퍼티 변경 요청 처리  

앞에서 생성한 소켓으로 프로퍼티 값의 변경 요청 메시지가 수신되면 **handle_property_set_fd() 함수가 호출**된다.  

```
void handle_property_set_fd(int fd)  
{  
    /* Check socket options here */  
    if (getsockopt(s, SOL_SOCKET, SO_PEERCRED, &cr, &cr_size) < 0) { // 소켓으로 부터 값을 얻어온다.  
        close(s);  
        ERROR("Unable to recieve socket options\n");  
        return;  
    }  
    :  
    :  
    switch(msg.cmd) {  
    case PROP_MSG_SETPROP:  
        if(memcmp(msg.name,"ctl.",4) == 0) { // "ctl"은 시스템 프로퍼티 값을 변경하는 것이 아니라, 프로세스의 시작, 종료를 요청하는 메시지.  
            if (check_control_perms(msg.value, cr.uid, cr.gid)) { // check_control_perms() 함수를 이용하여 접근 권한 검사. (system server, root, 해당 프로세스만 종료하거나 시작할 수 있음)  
                handle_control_message((char*) msg.name + 4, (char*) msg.value);  
            } else {  
                ERROR("sys_prop: Unable to %s service ctl [%s] uid: %d pid:%d\n", msg.name + 4, msg.value, cr.uid, cr.pid);  
            }  
        } else { // 시스템의 프로퍼티를 변경하는 데 사용.  
            if (check_perms(msg.name, cr.uid, cr.gid)) { // 접근 권한은 check_perms() 함수를 호출하여 검사한다.  
                property_set((char*) msg.name, (char*) msg.value); // 프로퍼티 값을 변경한다.  
            }  
        }  
    }  
}  
```

프로퍼티 값을 변경하고, 문제가 없다면 **property_changed() 함수가 호출**된다.  
