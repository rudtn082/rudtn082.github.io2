---
layout: post
title: "[인사이드 안드로이드] 챕터 6 - 안드로이드 서비스 개요"
description:
headline:
modified: 2019-05-06
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


## Chapter6 - 안드로이드 서비스 개요  


---------------------------------------


### 예제 프로그램 : 안드로이드 서비스 동작 이해  

안드로이드에서 서비스는 UI 없이 주기적으로 특정한 일을 수행하는 백그라운드 프로세스를 가리킨다. 따라서 안드로이드 프로그램을 작성할 때 개발자가 적절한 애플리케이션 서비스를 직접 구현해서 적용한다면 더 반응성이 좋은 애플리케이션을 개발할 수 있다.  

간단한 예제를 통해 안드로이드 서비스의 동작 방식을 알아보기 위해 ApiDemo 샘플 예제의 Alarm Service 코드를 본다.  

이 프로그램은 크게 메인 액티비티와 두개의 시스템 서비스, AlarmService_Service라는 애플리케이션 서비스로 구성돼 있다.  
(1) 메인 액티비티 상에서 Start 버튼을 누르면 시스템 서비스인 Alarm Service에 30초마다 사용자가 작성한 애플리케이션 서비스인 AlarmService_Service를 실행해 달라고 요청한다.  
(2) Alarm Service는 사용자의 요청에 따라 30초마다 AlarmService_Service라는 애플리케이션 서비스를 실행한다.  
```
private OnClickListener mStartAlarmListener = new OnClickListener() {
    public void onClick(View v) {
        long firstTime = SystemClock.elapsedRealtime();

        AlarmManager am = (AlarmManager)getSystemService(ALARM_SERVICE);
        am.setRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,
                        firstTime, 30*1000, mAlarmSender); // (1, 2)
    }
};
```


(3) AlarmService_Service는 실행되자마자 시스템 서비스인 Notification Service에 AlarmService_Service 서비스가 시작됐음을 알리는 문자열 출력을 요청한다.  
(4) Notification Service는 AlarmService_Service에게서 전달받은 문자열을 화면 상단의 상태 표시줄에 출력한다.  
```
public void onCreate() {
    NotificationManager mNM = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);

    showNotification(); // (3, 4)

    Thread thr = new Thread(null, mTask, "AlarmService_Service");
    thr.start();
}
```


(5) AlarmService_Service는 시작된 후 15초 후에 종료되며, 종료된 사실을 메인 액티비티에 알리기 위해 토스트(Toast) 메시지를 출력한다.  
```
Runnable mTask = new Runnable() { // (5)
    public void run() {
        long endTime = System.currentTimeMillis() + 15*1000;
        while (System.currentTimeMillis() < endTime) {
            synchronized (mBinder) {
                try {
                    mBinder.wait(endTime - System.currentTimeMillis());
                } catch (Exception e) {
                }
            }
        }

        AlarmService_Service.this.stopSelf();
    }
};


public void onDestroy() {
    mNM.cancel(R.string.alarm_service_started);

    Toast.makeText(this, R.string.alarm_service_finished, Toast.LENGTH_SHORT).show();
}
```


### 안드로이드 서비스 분류  

안드로이드 서비스는 다음과 같이 크게 프레임워크에서 기본적으로 제공하는 **시스템 서비스**와 애플리케이션 개발자가 Service 클래스를 상속해서 구현한 **애플리케이션 서비스**로 구분할 수 있다.  

![service1](/images/post/service1.png "service1")  

### 안드로이드 애플리케이션 서비스  

애플리케이션 서비스는 안드로이드 SDK의 Service 클래스를 확장한 클래스의 인스턴스로 UI 없이 주기적으로 특정한 일을 수행하는 백그라운드 프로세스를 가리킨다.

애플리케이션 개발자는 서비스를 두 가지 방법으로 이용할 수 있다.  
* 서비스 시작, 종료  
* 바인딩을 통한 서비스 원격 제어(서비스를 원격 제어할 수 있게 서비스에 연결하는 것)  

애플리케이션이 서비스를 생성하려면 startService(), bindService()같은 API를 상황에 맞게 이용하면 된다. 백그라운드에서 특정 동작을 하는 서비스를 실행할 때는 **startService()**, 서비스에 바인딩해서 서비스가 제공하는 인터페이스를 통해 서비스를 제어하고 싶다면 **bindService()**를 통해 서비스를 생성한다. 두 서비스의 생명 주기는 약간 다른 것을 확인할 수 있다.  

![service2](/images/post/service2.png "service2")  

startService()나 bindService()로 시작된 서비스 모두 onCreate()와 onDestroy() 콜백 메서드가 호출된다. onCreate() 메서드는 서비스가 처음 생성될 때 호출되며 일반적으로 서비스를 초기화하는 코드가 포함된다. onDestroy()는 서비스가 종료되기 직전에 호출되는데, 이때 서비스가 사용한 리소스를 모두 해제해야 한다.  

##### startService()  
onStartCommand()는 그림에서 볼 수 있듯이 오직 startService()에 의해 시작된 서비스에서 onCreate() 메서드 다음으로 호출된다. startService() 메서드의 첫 번째 인자로 넘어온 인텐트가 onStartCommand()의 첫 번째 인자로 그대로 전달되는데, 이때 인텐트에는 주로 실행할 서비스에 대한 정보가 포함된다. 이후 onStartCommand()에서는 일반적으로 인텐트에서 넘어온 인자가 있다면 이를 처리하거나 실질적인 백그라운드 작업을 처리하는 스레드를 실행한다.  

##### bindService()  
bindService()에 의해 시작된 서비스에서 onBind()는 클라이언트가 서비스에 바인딩하려고 할 떄 호출된다.(아직 서비스가 생성되지 않았다면 그 전에 onCreate()를 먼저 호출한다) 서비스의 onBind() 콜백에서는 바인딩할 클라이언트를 위해 해당 서비스와 연결 가능한 객체를 제공한다. 따라서 바인딩이 끝나면 클라이언트는 이 객체를 통해 서비스를 원격 제어할 수 있다.  


#### 애플리케이션 서비스의 분류  

애플리케이션 서비스를 로컬 서비스와 리모트 서비스로 구분한다. 이를 구분하는 기준은 서비스와 이를 생선한 서비스 클라이언트(보통 액티비티)가 동일한 프로세스에서 동작하고 있는지 여부다.  

![service3](/images/post/service3.png "service3")  

위의 그림과 같이 생성된 서비스가 자신과 동일한 프로세스에서 실행되는 경우 **로컬 서비스**라 부른다. 로컬 서비스는 자신을 생성한 애플리케이션 내에서만 사용될 수 있으며 애플리케이션이 종료하면 함께 종료한다.  
**리모트 서비스**는 자신을 생성한 액티비티와는 별개의 독립적인 프로세스 위에서 동작하기 때문에 메인 애플리케이션이 종료하더라도 계속 동작한다. **잘못 구현된 리모트 서비스는 프로그램이 종료하더라도 시스템 자원(배터리 등)을 비효율적으로 소모할 수 있기 때문에 설계에 신중을 기해야 한다.  

로컬 서비스와 리모트 서비스와의 가장 큰 차이는 서비스 제어를 위한 바인딩 방법이다.  
로컬 서비스의 경우는 서비스와 서비스를 이용하는 클라이언트 프로그램이 동일 프로세스에서 동작하기 때문에 로컬 서비스 바인딩은 클라이언트 프로그램이 로컬 서비스의 레퍼런스만 얻으면 된다.  
리모트 서비스의 경우는 액티비티와 자신이 모두 별개의 프로세스에서 동작하므로 (다음 장에서 배울)IPC 메커니즘을 이용해야 한다.  

로컬 서비스와 리모트 서비스의 바인딩 과정이 어떻게 다른지를 예제를 분석하며 알아보겠다.  

##### 로컬 서비스  

Local Service Binding 예제는 크게 서비스를 나타내는 LocalService.java와 서비스를 이용하는 액티비티인 LocalServiceActivities.java로 구성돼 있다.  
Bind Service 버튼을 누르면 doBindService()가 호출된다.  
```
private OnClickListener mBindListener = new OnClickListener() {
    public void onClick(View v) {
        doBindService();
    }
};
```

이 메서드에서 내부적으로 bindService() API를 사용해서 LocalService 바인딩을 시도한다.  
*bindService(LocalService 실행하기 위한 인텐트, 서비스와의 바인딩 연결을 처리할 객체, 바인딩할 서비스가 없는 경우 자동 서비스 생성 플래그)*  
```
void doBindService() {
    if (bindService(new Intent(Binding.this, LocalService.class),
        mConnection, Context.BIND_AUTO_CREATE)) {
        mShouldUnbind = true;
    } else {
        Log.e("MY_APP_TAG", "Error: The requested service doesn't " +
                "exist, or this client isn't allowed access to it.");
    }
}
```

바인딩할 서비스가 생성됐으므로 안드로이드는 바인딩 처리를 위해 서비스의 onBind() 콜백 메서드를 호출한다.  
onBind 메서드는 액티비티가 LocalService 자신과 연결할 수 있게 LocalBinder 객체를 반환한다.  
```
public IBinder onBind(Intent intent) {
        return mBinder;
    }

private final IBinder mBinder = new LocalBinder();

public class LocalBinder extends Binder {
    LocalService getService() {
        return LocalService.this;
    }
}
```

바인딩을 처리할 객체를 제대로 생성했다면(LocalBinder), 안드로이드 프레임워크는 서비스 클라이언트 측 메서드를 호출한다.  
이 때 LocalBinder 객체의 getService() 메서드를 호출해서 바인딩하려고 했던 LocalService 객체의 레퍼런스 값을 구한다.  
이렇게 구한 LocalBinder 객체의 레퍼런스 값을 mBoundService 멤버 필드에 저장하면 서비스 바인딩이 마무리된다.  
```
private ServiceConnection mConnection = new ServiceConnection() {
    public void onServiceConnected(ComponentName className, IBinder service) {
        mBoundService = ((LocalService.LocalBinder)service).getService();
        Toast.makeText(Binding.this, R.string.local_service_connected,
            Toast.LENGTH_SHORT).show();
    }

    public void onServiceDisconnected(ComponentName className) {
        mBoundService = null;
        Toast.makeText(Binding.this, R.string.local_service_disconnected,
            Toast.LENGTH_SHORT).show();
        }
    };
```

서비스가 바인딩 되고 나면 액티비티는 해당 서비스가 가진 모든 메서드와 멤버 필드 값을 mBoundService 멤버 필드를 통해 접근할 수 있게 된다.  

##### 리모트 서비스  

Remote Service Binding 예제는 Local Service Binding 과는 다르게 액티비티나 서비스 파일 이외에 ISecondary.aidl이라는 AIDL 파일과 이 파일에 의해 자동 생성된 ISecondary.java 파일이 추가된 것을 알 수 있다.  
*ISecondary.aidl은 액티비티와 서비스의 통신을 위한 인터페이스 정의*
*ISecondary.java는 안드로이드가 자동 생성하며 인터페이스를 기반으로 액티비티와 서비스가 서로 통신할 수 있게 마샬링/언마샬링을 수행*
*마샬링 - 객체의 메모리 구조를 저장이나 전송을 위해서 적당한 자료형태로 변형*

ISecondary.aidl을 통해 자동으로 생성된 ISecondary.java는 서비스 클라이언트와 리모트 서비스 간의 ISecondary 인터페이스에 기반한 바인더 IPC 연결을 설정한다. 즉, 이 코드를 통해서 RemoteServiceBinding 액티비티는 RemoteService 서비스가 제공하는 ISecondary 인터페이스에 포함된 getPid() 메서드를 마치 해당 클래스에 포함된 매서드를 호출하듯 호출할 수 있다.  

Bind Service 버튼을 누르면 액티비티와 동립적인 프로세스에서 RemoteService가 실행된다. doBindService()가 호출된다.  
