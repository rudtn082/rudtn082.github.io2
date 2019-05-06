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
