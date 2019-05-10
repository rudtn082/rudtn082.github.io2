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

**② Surface Flinger 서비스**  

Surface Flinger는 다양한 애플리케이션에서 사용중인 **Surface를 조합해 프레임 버퍼 장치로 렌더링**해주는 서비스다.  

![service6](/images/post/service6.png "service6")  


##### 자바 시스템 서비스  

자바 시스템 서비스는 안드로이드 부팅 시 SystemServer라는 시스템 프로세스에 의해 일괄적으로 실행되며, ① 코어 플랫폼 서비스와 ② 하드웨어 서비스로 나뉜다.  

**① 코어 플랫폼 서비스**  

코어 플랫폼 서비스는 일반적으로 안드로이드 애플리케이션과 직접 상호작용은 하지 않지만 **안드로이드 프레임워크가 동작하는 데 필수적인 서비스**를 말한다.  

| 코어 플랫폼 서비스 | 기능 |
| :---: | :---: |
| `Activity Manager Service` | 모든 액티비티에 대한 라이프 사이클 및 액티비티 스택 관리 |
| `Window Manager Service` | Surface Flinger 위에 위치하며, 기기 화면에 그릴 내용을 Surface Flinger로 전달 |
| `Package Manager Service` | apk파일의 정보를 로딩, 시스템에 어떤 패키지가 설치되고 로딩돼 있는지에 대한 정보 제공 |

**② 하드웨어 서비스**  

저수준 하드웨어 제어를 위한 API를 제공하는 서비스를 말한다.  

| 하드웨어 서비스 | 기능 |
| :---: | :---: |
| `Alarm Manager Service` | 타이머처럼 특정 시간 후에 애플리케이션을 실행하는 등의 동작을 수행 |
| `Connectivity Service` | 네트워크의 현재 상태에 대한 정보를 제공 |
| `Location Service` | 단말의 현재 위치 정보를 제공 |
| `Power Service` | 장치의 전원 관리를 제어 |
| `Sensor Service` | 안드로이드에 설치된 각종 센서(마그네틱 센서, 가속도 센서 등)의 센싱 값을 제공 |
| `Telephony Service` | 전화기의 상태나 전화 서비스에 대한 정보를 제공 |
| `Wifi Service` | AP 검색이나 연결 리스트 관리 등 무선랜 연결을 제어 |


**자바 시스템 서비스 이용**  

프레임워크 내부에서나 안드로이드 애플리케이션에서 이러한 자바 시스템 서비스를 이용하려면 **각 서비스와 통신 가능한 Local Manager 객체**를 이용해야 한다.  


### 시스템 서비스의 실행  

안드로이드 애플리케이션에서 애플리케이션 서비스를 실행할 경우 일반적으로 startService() API함수를 사용한다. 그러나 시스템 서비스는 애플리케이션 서비스와 달리 서비스를 직접 실행할 필요가 없이 getSystemService()를 이용해서 바로 이용할 수 있다. **시스템 서비스는 init 프로세스에 의해 안드로이드 부팅 과정에서 미리 실행되기 때문이다.**  

![service7](/images/post/service7.png "service7")  

#### 미디어 서버의 실행 코드 분석  

##### init 프로세스로부터 실행  

미디어 서버(Media Server)는 Audio Flinger, Media Player Service, Camera Service, Audio Policy Service 등의 네이티브 시스템 서비스를 실행하는 시스템 프로세스로서 init 프로세스에 의해 실행된다.  
```
service media /system/bin/mediaserver
    user media
    group system audio camera graphics inet net_bt net_bt_admin
```


##### 네이티브 서비스 인스턴스 생성 및 초기화  

다음은 미디어 서버에 포함된 main() 함수의 주요 부분이다. 여기서는 각 네이티브 서비스에 대한 인스턴스를 생성하고 초기화하는 작업을 수행한다.  
```
int main(int argc, char** argv)
{
  :
  AudioFlinger::instantiate();
  MediaPlayerService::instantiate();
  CameraService::instantiate();
  AudioPolicyService::instantiate();
  :
}
```

##### 각 시스템 서비스별 초기화 코드 살펴보기  

시스템 서비스는 프레임워크 내에서 다른 모듈과 통신할 때 바인더 IPC를 사용한다.  
*(다음장, 애플리케이션과 같은 서비스 이용자가 시스템 서비스를 이용할 수 있게 서비스 제공자의 정보를 컨텍스트 매니저에 등록)*  

다음은 네이티브 서비스의 초기화 코드를 모아놓은 것이다. new 연산자를 통해 서비스 인스턴스를 생성한 다음 addService() 함수를 이용해서 컨텍스트 매니저에게 각 서비스를 등록한다.  
여기서 defaultServiceManager() 함수는 컨텍스트 매니저와 바인더 통신을 하는 일종의 프록시 객체인 서비스 매니저를 반환한다. (자세한 내용은 8장)  

```
// Audio Flinger 초기화 코드
void AudioFlinger::instantiate() {
    defaultServiceManager()->addService(
            String16("media.audio_flinger"), new AudioFlinger());
}

// Media Player Service 초기화 코드
void MediaPlayerService::instantiate() {
    defaultServiceManager()->addService(
            String16("media.player"), new MediaPlayerService());
}

// Camera Service 초기화 코드
void CameraService::instantiate() {
    defaultServiceManager()->addService(
            String16("media.camera"), new CameraService());
}

// Audio Policy Service 초기화 코드
void AudioPolicyService::instantiate() {
    defaultServiceManager()->addService(
            String16("media.audio_policy"), new AudioPolicyService());
}
```

미디어 서버는 각 서비스의 인스턴스를 생성한 다음, 생성된 서비스를 컨텍스트 매니저에 등록하는 것이 전부다.  

![service8](/images/post/service8.png "service8")  



#### 시스템 서버의 실행 코드 분석  

시스템 서버는 자바 프로세스이므로 Zygote로부터 생성된다. 시스템 서버가 어떻게 시스템 서비스를 생성하는지 코드를 바탕으로 알아보자.  


##### Zygote 프로세스로부터 생성  

Zygote는 맨 처음 생성되는 달빅 가성 머신 기반의 자바 프로세스로서 네이티브 시스템 서비스인 Surface Flinger를 비롯해 다양한 자바 시스템 서비스를 실행하는 역할을 한다.  

Zygote 실행 시에 -start-system-server라는 옵션이 정의된 것을 확인할 수 있다. 이 옵션은 Zygote에서 시스템 서버를 생성하라고 요청하는 역할을 한다.  
```
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
    class main
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart media
    onrestart restart netd
```

##### android_servers 라이브러리 로드  

다음은 SystemServer의 main() 메서드를 나타내는 코드다.  
android_servers 라이브러리를 로드하고 init1()메서드를 호출한다. init1() 메서드는 JNI를 통해 다음 system_init() 네이티브 함수를 호출한다.
```
public class SystemServer {
  native public static void init1(String[] args);

  public static void main(String[] args) {
    System.loadLibrary("android_servers");
    init1(args);
  }
}
```

system_init() 함수의 주된 기능은 Surface Flinger 네이티브 시스템 서비스를 초기화하는 것이다.  
```
extern "C" status_t system_init()
{
    SurfaceFlinger::instantiate(); // JNI를 이용해 Surface Flinger 서비스 실행

    AndroidRuntime* runtime = AndroidRuntime::getRuntime();
    JNIEnv* env = runtime->getJNIEnv();

    jclass clazz = env->FindClass("com/android/server/SystemServer");
    jmethodID methodId = env->GetStaticMethodID(clazz, "init2", "()V"); // SystemServer 클래스의 init2() 메서드를 호출한다.

    return NO_ERROR;
}
```

##### 자바 시스템 서비스 초기화 및 등록  

SystemServer에서 Surface Flinger를 초기화하고 나면 init2() 메서드가 호출된다. 이 메서드는 Entropy 서비스부터 AppWidget 서비스까지 **안드로이드의 모든 자바 시스템 서비스를 생성하고 초기화하는 역할**을 한다.  
init2() 메서드에서는 ServerThread를 생성한 다음 실행한다.  
이렇게 실행된 자바 시스템 서비스 역시 네이티브 시스템 서비스처럼 다른 모듈이 서비스를 활용할 수 있게끔 해당 서비스를 컨텍스트 매니저에 등록해야 한다. 자바 시스템 서비스는 ServiceManager 클래스의 **addService() 정적 메서드를 이용해서 컨텍스트 매니저에 자신을 등록**한다.  
```
public static final void init2() {
    Thread thr = new ServerThread();
    thr.setName("android.server.ServerThread");
    thr.start();
}

class ServerThread extends Thread {
  :
  Slog.i(TAG, "Entropy Mixer");
  ServiceManager.addService("entropy", new EntropyMixer());

  Slog.i(TAG, "Power Manager");
  power = new PowerManagerService();
  ServiceManager.addService(Context.POWER_SERVICE, power);

  Slog.i(TAG, "Activity Manager");
  context = ActivityManagerService.main(factoryTest);

  Slog.i(TAG, "Telephony Registry");
  ServiceManager.addService("telephony.registry", new TelephonyRegistry(context));

  :
  :

  Slog.i(TAG, "Backup Service");
  ServiceManager.addService(Context.BACKUP_SERVICE, new BackupManagerService(context));

  Slog.i(TAG, "AppWidget Service");
  appWidget = new AppWidgetService(context);
  ServiceManager.addService(Context.APPWIDGET_SERVICE, appWidget);

  :
  :
}

```

![service9](/images/post/service9.png "service9")  


### 안드로이드 서비스 프레임워크와 바인더 드라이버 개요 및 용어 정리  

#### 주요 용어 정리  

**서비스 서버** : 시스템 서비스를 실행하는 프로세스로서 앞에서 설명한 시스템 서버나 미디어 서버가 여기에 해당한다.  
**서비스 클라이언트** : 시스템 서비스를 사용하는 프로세스를 일컫는다.  
**컨텍스트 매니저** : 시스템 서비스를 관리하는 안드로이드 시스템 프로세스로서 시스템에 설치돼 있는 각종 시스템 서비스의 위치 정보인 핸들을 관리한다. 이러한 핸들은 바인더 IPC의 목적지 주소를 지정하는 데 사용된다.  
**서비스 프레임워크** : 서비스 매니저를 포함해서 서비스 사용자와 시스템 서비스간의 RPC 동작에 필요한 공통적인 클래스가 정의돼 있다.  
**서비스 인터페이스** : 서비스 사용자와 시스템 서비스 간에 미리 정해진 인터페이스로서 시스템 서비스는 해당 인터페이스에 맞게 스텁 함수를 구현해서 해당 서비스를 제공해야 하고, 반대로 서비스 사용자 역시 해당 인터페이스에 맞게 서비스를 호출해야 한다.  
**서비스 사용자** : 서비스 클라이언트 프로세스 내에서 실제 서비스를 이용하는 모듈이다.  
**서비스** : 서비스 인터페이스에 정의된 기능을 서비스 스텁 함수로 구현해서 실제 서비스의 기능을 제공하는 모듈을 의미한다.  
**서비스 프록시** : RPC 수행 시 데이터 마샬링을 수행하는 객체이며 서비스 인터페이스별로 존재한다. 서비스 인터페이스에 정의된 함수별로 각각 데이터 마샬링을 수행하는 서비스 프록시 함수를 제공한다.  
**서비스 스텁** : RPC 수행 시 데이터 언마샬링을 수행하는 객체이며, 이 객체 역시 서비스 인터페이스별로 존재한다. 수신된 데이터를 언마샬링해서 연관된 서비스 스텁 함수를 호출한다.  
**바인더 드라이버** : 바인더는 안드로이드에서 IPC를 지원하는 데 사용되는 메커니즘으로 안드로이드 리눅스 커널의 디바이스 드라이버 형태로 포함돼 있다.  
**바인더 IPC** : 안드로이드에서 바인더 드라이버를 통한 프로세스간의 데이터 전달 방식을 말한다.  
**바인더 IPC 데이터** : 서비스 프레임워크와 바인더 드라이버 사이에 사용되는 데이터 포맷  
**바인더 RPC** : 서비스 사용자가 서비스에서 제공하는 특정 서비스 인터페이스 기반의 함수를 마치 자신의 로컬 함수 호출하듯이 원격으로 처리하는 동작을 말한다. 바인더 RPC는 내부적으로는 바인더 IPC 메커니즘 기반으로 동작한다.  
**바인더 RPC 데이터** : 서비스 사용자와 서비스 간의 바인더 RPC를 수행하는 데 사용되는 데이터

다음 그림은 서비스 동작 원리를 일반화한 것으로서 Foo라는 시스템 서비스에서 제공하는 foo() 메서드를 서비스 사용자가 RPC 형태로 호출하는 모습을 보여준다.  

![service10](/images/post/service10.png "service10")  

서비스 사용자는 foo() 프록시 함수를 호출해서 Foo 서비스를 이용하기 위한 인자로 구성된 바인더 RPC 데이터를 전달한다. 바인더 PRC 데이터는 마샬링을 거쳐 서비스 프레임워크를 통해 바인더 IPC 데이터로 생성된 다음 바인더 드라이버를 통해 서비스 서버 측에 전송된다.  

서비스 서버 측에서 수신된 바인더 IPC 데이터는 서비스 프레임워크를 거치면서 언마샬링된 다음, 서비스 스텁의 onTransact() 함수에 전송된다. 서비스 스텁은 해당 바인더 IPC 데이터 안에 포함된 RPC 코드를 통해 Foo 서비스의 foo() 서비스 스텁 함수에 대한 바인더 RPC임을 판단한다. 수신된 바인더 IPC 데이터에 포함된 바인더 RPC 데이터를 인자로 해서 foo() 서비스 스텁 함수를 호출한다.  
