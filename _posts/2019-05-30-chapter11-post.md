---
layout: post
title: "[인사이드 안드로이드] 챕터 11 - 자바 시스템 서비스 동작 분석"
description:
headline:
modified: 2019-05-30
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


## Chapter11 - 자바 시스템 서비스 동작 분석  


---------------------------------------


### 액티비티 매니저 서비스  

액티비티 매니저 서비스는 자바 시스템 서비스의 일종인 코어 플랫폼 서비스로서 안드로이드 애플리케이션 컴포넌트인 액티비티, 서비스, 브로드캐스트 리시버등을 생성하고, 이들의 생명주기를 관리하는 역할을 한다.  

ApiDemos 예제 코드에 들어있는 Remote Service Controller 애플리케이션을 토대로 액티비티 매니저 서비스가 안드로이드의 애플리케이션 서비스를 어떻게 생성하고 해당 서비스의 생명주기를 어떻게 제어하는지 알아본다.  

![JS1](/images/post/JS1.png "JS1")  

Remote Service Controller 예제 애플리케이션은 별도의 프로세스를 통해 리모트 서비스를 실행하는 예제로 ⑴ 'Start Service' 버튼을 누르면 ⑵ RemoteService가 시작되는 간단한 프로그램이다.  

![JS2](/images/post/JS2.png "JS2")  

⑴ 안드로이드 애플리케이션은 startService()나 bindService() API를 통해 애플리케이션 서비스를 생성한다.  
⑵ 애플리케이션으로부터 startService()를 통해 서비스 실행 요청을 받은 액티비 매니저 서비스는 요청받은 서비스 클래스(RemoteService.class)를 바로 로드하는 것이 아니라 Zygote에게 서비스를 실행시키기 위한 ActivityThread 생성을 요청한다.  
⑶ 액티비티 매니저 서비스로부터 ActivityThread 실행을 요청받은 Zygote는 새로운 프로세스를 생성한 다음 그위에 ActivityThread 클래스를 로딩한다.  
⑷ ActivityThread에게 RemoteService 서비스의 생성을 요청한다.  
⑸ ActivityThread는 RemoteService를 실행한다.  

이렇듯 안드로이드의 액티비티 매니저 서비스는 애플리케이션 서비스를 포함한 액티비티, 브로드캐스트 리시버 같은 안드로이드 애플리케이션 컴포넌트를 생성하는 중요한 역할을 하는 시스템 서비스다.  


### 액티비티 매니저 서비스를 통한 서비스 생성 코드 분석  

액티비티가 startService() API 메서드를 호출할 경우 액티비티 매니저 서비스가 어떻게 애플리케이션 서비스를 생성하는지 소스 코드를 바탕으로 자세히 살펴보자  

#### Controller 액티비티 - startService() 메서드 호출  

⑴과 같이 'Start Servcie' 버튼을 누르면 코드 11-1의 이벤트 핸들러가 호출된다. 이 경우 단순히 인텐트(intent)를 인자로 startService() API 메서드를 호출한다.  

```
private OnClickListener mStartListener = new OnClickListener() {
    public void onClick(View v) {
        startService(new Intent("com.example.android.apis.app.REMOTE_SERVICE"));
    }
};
```

#### 액티비티 매니저 서비스의 startService() 메서드 호출 과정(바인더 RPC 활용)  

액티비티에서 호출한 startService() API는 서비스 생성 및 실행과 관련된 내용을 액티비티 매니저 서비스에 요청하는 기능만 수행할 뿐 실제 구현은 액티비티 매니저 서비스에 속한, 동일한 이름을 가진 startService() 스텁 메서드에 들어 있다.  

즉, 액티비티에서 호출한 startService() API는 자바 서비스 프레임워크 기반에서 바인더 RPC 형태로 액티비티 매니저 서비스에서 제공하는 startService() 스텁 메서드를 호출하게 되는 것이다.  

![JS3](/images/post/JS3.png "JS3")  

⑴ Controller 액티비티 - ActivityManagerProxy 객체의 startService() 프록시 메서드를 호출  
⑵ ActivityManagerProxy 객체 - 자바 서비스 프레임워크를 통해 ActivityManagerNative 객체에 START_SERVICE_TRANSACTION RPC 코드와 바인더 RPC 데이터를 전송  
⑶ ActivityManagerNative 객체 - ActivityManagerService에 포함된 startService() 스텁 메서드를 호출  

##### ⑴ Controller 액티비티  

```
public class ContextWrapper extends Context {
    Context mBase;

    public ComponentName startService(Intent service) {
        return mBase.startService(service); // ContextImpl 객체의 startService() 메서드 호출
    }
}
```

ContextWrapper는 context 추상 클래스를 확장한 클래스로 멤버 변수 mBase에 저장된 context 객체를 래핑(wrapping)하는 역할을 한다. 현재 ContextWrapper 객체는 그림에서 볼 수 있듯이 Controller 액티비티의 ContextImpl 객체를 래핑하고 있다.  

```
public ComponentName startService(Intent service) {
    ComponentName cn = ActivityManagerNative.getDefault().startService(
                          mMainThread.getApplicationThread(), service,
                          service.resolveTypeIfNeeded(getContentResolver()));

    return cn;
}
```

다음 그림은 위 코드의 핵심 부분인 ActivityManagerNative.getDefault().startService()의 과정을 나타낸다. 아래 그림에서도 확인할 수 있듯이 ActivityManagerNative.getDefault() 함수는 결국 ActivityManagerProxy 객체를 반환하는데 이 객체는 액티비티 매니저 서비스가 제공하는 IActivityManager 서비스 인터페이스 기반의 메서드들을 바인더 RPC 를 통해서 호출하는 역할을 한다. 따라서 액티비티 측에서는 이 객체를 통해 startService() 스텁 메서드처럼 액티비티 매니저 서비스가 제공하는 IActivityManager 인터페이스에 포함된 다양한 메서드를 로컬 함수를 호출하듯 자유롭게 이용할 수 있다.  

![JS4](/images/post/JS4.png "JS4")  

따라서 ActivityManagerNative.getDefault().startService() 메서드는 ActivityManagerProxy 클래스의 startService() 프록시 메서드를 호출한다. 이는 결국 ActivityManagerService의 startService() 스텁 메서드를 원격으로 호출하는 역할을 수행한다.  

```
public ComponentName startService(IApplicationThread caller, Intent service, String resolvedType)
```

startService() 메서드의 주요 인자를 간단히 살펴보면 첫 번째 인자인 caller는 IApplicationThread 타입의 변수로서 액티비티 매니저 서비스로부터 전송된 IApplicationThread 서비스 인터페이스 기반의 바인더 RPC 를 처리하는 역할을 수행한다.  

##### ⑵ ActivityManagerProxy 객체 - startService() 프록시 메서드 처리  


```
public ComponentName startService(IApplicationThread caller, Intent service,
        String resolvedType) throws RemoteException
{
    Parcel data = Parcel.obtain();
    Parcel reply = Parcel.obtain();

    // 전송 data 생성 (인자값을 data에 저장)
    data.writeInterfaceToken(IActivityManager.descriptor);
    data.writeStrongBinder(caller != null ? caller.asBinder() : null);
    service.writeToParcel(data, 0);
    data.writeString(resolvedType);

    // 바인더 RPC 데이터 전송
    mRemote.transact(START_SERVICE_TRANSACTION, data, reply, 0);
    reply.readException();
    ComponentName res = ComponentName.readFromParcel(reply);
    data.recycle();
    reply.recycle();

    return res;
}
```

위 코드에서 mRemote.transact() 메서드는 자바 객체에서 바인더 RPC 데이터를 전송하는 데 쓰인다. ActivityManagerProxy 객체는 startService() 프록시 메서드의 인자로 전달된 caller, service, resolved에 들어 있는 값을 바인더 RPC 데이터를 저장하는 데 사용되는 Pacel 객체 변수인 data에 저장한다. 그런 다음 START_SERVICE_TRANSACTION 트랜잭션을 통해 저장한 data 값을 ActivityManagerNative 객체에 전달한다.  

##### ⑶ ActivityManagerNative 객체 - startService() 스텁 메서드 호출  

ActivityManagerNative 객체는 상대편인 ActivityManagerProxy 객체에게서 전달받은 RPC 코드를 토대로 액티비티 매니저 서비스에서 호출한 스텁 매서드를 파악한다. 여기서는 ActivityManagerProxy 객체가 START_SERVICE_TRANSACTION RPC코드를 전송했으므로 아래 코드에서 볼 수 있듯이 startService() 스텁 메서드가 호출돼야 한다.  

다음은 startService() 스텁 메서드에 전달해야 할 인자를 구해서 startService() 스텁 메서드를 실제로 호출하면 된다. 이를 위해 RPC 데이터를 언마샬링한 후, 액티비티 매니저 서비스의 startService() 스텁 메서드를 호출한다.  

```
public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
        throws RemoteException {
    switch (code) {
    case START_ACTIVITY_TRANSACTION:
    {
        data.enforceInterface(IActivityManager.descriptor);
        IBinder b = data.readStrongBinder();
        IApplicationThread app = ApplicationThreadNative.asInterface(b);
        Intent intent = Intent.CREATOR.createFromParcel(data);
        String resolvedType = data.readString();

        Uri[] grantedUriPermissions = data.createTypedArray(Uri.CREATOR);
        int grantedMode = data.readInt();
        IBinder resultTo = data.readStrongBinder();
        String resultWho = data.readString();    
        int requestCode = data.readInt();
        boolean onlyIfNeeded = data.readInt() != 0;
        boolean debug = data.readInt() != 0;
        String profileFile = data.readString();
        ParcelFileDescriptor profileFd = data.readInt() != 0
                ? data.readFileDescriptor() : null;
        boolean autoStopProfiler = data.readInt() != 0;
        int result = startActivity(app, intent, resolvedType,
                grantedUriPermissions, grantedMode, resultTo, resultWho,
                requestCode, onlyIfNeeded, debug, profileFile, profileFd, autoStopProfiler);
        reply.writeNoException();
        reply.writeInt(result);
        return true;
    }
    :
    return super.onTransact(code, data, reply, flags);
}
```

onTransact() 메서드의 역할은 아래 그림과 같이 ActivityManagerProxy의 startService() 프록시 메서드의 인자 값(caller, service, resolvedType)이 마샬링된 data 변수(Parcel 객체)를 바인더 RPC를 통해 수신한 다음, data 변수를 언마샬링하고 각 데이터를 별도의 변수에 저장하는 것이다. 그러고 나서 저장된 변수를 인자로 삼아 액티비티 매니저 서비스의 startService() 스텁 메서드를 호출하는 것이다.  

![JS5](/images/post/JS5.png "JS5")  

아래 코드에 대해 살펴보면,

```
IApplicationThread app = ApplicationThreadNative.asInterface(b);
```

ApplicationThreadNative.asInterface() 메서드는 ApplicationThreadNative 객체에 대응하는 ApplicationThreadProxy 객체를 생성한다. 그러고 나면 서비스 생성을 요청한 RemoteActivityController 액티비티와 액티비티 매니저 서비스 간에는 다음 그림과 같이 두 개의 바인더 연결이 성립된다. 액티비티는 IActivityManager 서비스 인터페이스 기반의 바인더 RPC를 통해 서비스 실행, 인텐트 송수신 등의 기능 수행을 요청할 수 있다. 반대로 액티비티 매니저 서비스는 IApplicationThread 인터페이스 기반의 바인더 RPC를 통해 자신과 연결된 애플리케이션을 제어할 수 있다.  

![JS6](/images/post/JS6.png "JS6")  

#### 액티비티 매니저 서비스 - startService() 스텁 메서드 실행  

이제 액티비티 매니저 서비스가 요청받은 서비스를 어떻게 실행하는지 살펴본다.  

```
public ComponentName startService(IApplicationThread caller, Intent service,
        String resolvedType, int userId) {

        final int callingPid = Binder.getCallingPid();
        final int callingUid = Binder.getCallingUid();
        final long origId = Binder.clearCallingIdentity();
        ComponentName res = mServices.startServiceLocked(caller, service,
                resolvedType, callingPid, callingUid, userId);
        return res;
    }
```

이 메서드에서 주로 하는 일은 startServiceLocked() 메서드를 호출하는 것이다.  

```
ComponentName startServiceLocked(IApplicationThread caller, Intent service, String
            resolvedType, int callingPid, int callingUid, int userId) {
    // retrieveServiceLocked() 메서드의 반환값인 ServiceLookupResult 구조체 변수 res로부터 ServiceRecord 값을 얻는다.
    ServiceLookupResult res =
            retrieveServiceLocked(service, resolvedType,
                    callingPid, callingUid, userId, true, callerFg);
    ServiceRecord r = res.record;
    return startServiceInnerLocked(smap, service, r, callerFg, addToStarting);
}
```

이 메서드에서 주로 하는 일은 ServiceRecord 값을 얻는 것이다. ServiceRecord는 안드로이드 애플리케이션 서비스에 대한 각종 정보(서비스 패키지명과 위치, 권한, 서비스 프로세스 정보, 실행 통계 정보 등)가 담긴 클래스이다.  

이렇게 구한 ServiceRecord 객체를 bringUpServiceLocked() 메서드의 첫 번째 인자로 전달한다.  

```
private final String bringUpServiceLocked(ServiceRecord r, int intentFlags,
            boolean execInFg, boolean whileRestarting) {

    final String appName = r.processName;
    ProcessRecord app = getProcessRecordLocked(appName, r.appInfo.uid);

    if (app != null && app.thread != null) {
          // 기존 프로세스 영역 내에서 서비스를 실행함.
          realStartServiceLocked(r, app, execInFg);
          return true;
    }

    startProcessLocked(appName, r.appInfo, true, intentFlags, "service", r.name, false);
    mPendingServices.add(r);

    return true;
}
```

bringUpServiceLocked() 메서드는 ServiceRecord 객체를 참조해서 해당 서비스가 실행된 프로세스 이름과 uid를 통해 ProcessRecord 객체가 존재하는지 검사한다. 이를 위해 getProcessRecordLocked() 메서드가 호출된다.  

```
final ProcessRecord startProcessLocked(String processName, ApplicationInfo info,
            boolean knownToBeDead, int intentFlags, String hostingType, ComponentName hostingName,
            boolean allowWhileBooting) {
                // ProcessRecord를 새로 생성
                app = getProcessRecordLocked(null, info, processName);
                mProcessNames.put(processName, info.uid, app);
                startProcessLocked(app, hostingType, hostingNameStr);

                return (app.pid != 0) ? app : null;
            }

private final void startProcessLocked(ProcessRecord app, String hostingType, String hostingNameStr) {
  int uid = app.info.uid;
  int[] gids = null;
  int debugFlags = 0;

  gids = mContext.getPackageManager().getPackageGids(app.info.packageName);

  int pid = Process.start("android.app.ActivityThread", null, uid, uid, gids, debugFlags, null);

  // 액티비티 매니저 서비스의 mPidsSelfLocked 해시에 생성된 프로세스의 pid 값을 key로 해서 ProcessRecord 객체를 저장함.
  this.mPidsSelfLocked.put(pid, app);
}
```

ActivityManagerService 클래스 코드에는 두 개의 startProcessLocked() 메서드가 존재한다.  

첫 번째 startProcessLocked() 메서드의 역할은 리모트 서비스를 실행하기 위해 새로 생성할 프로세스 정보를 포함하는 ProcessRecord 객체를 만들고, 이를 mProcessNames 큐에 삽입하는 것이다. 이 과정이 성공적으로 끝나면 두 번째 startProcessLocked() 메서드를 호출한다.  

두 번째 startProcessLocked() 메서더의 역할은 Process 클래스의 start() 메서드를 통해 Zygote에게 android.app.ActivityThread 프로세스 생성을 요청하는 것이다.  


#### ActivityThread 클래스의 main() 메서드 실행  

지금부터는 Zygote가 서비스 실행을 위해 액티비티 매니저 서비스가 요청한 ActivityThread 클래스를 새로운 프로세스 상에서 어떻게 실행하는지 알아본다.  

클래스의 실행을 요청받으면 새로운 프로세스를 생성하고 그 위에 해당 클래스를 로드한 후 해당 클래스의 main() 메서드를 호출한다.  

```
public final class ActivityThread extends ClientTransactionHandler {
    // ApplicationThread() 생성
    final ApplicationThread mAppThread = new ApplicationThread();

    public static final void main(String[] args) {
            Process.setArgV0("<pre-initialized>");
            Looper.prepareMainLooper(); // Looper.prepareMainLooper() 메서드를 이용해서 메시지 큐를 생성
            ActivityThread thread = new ActivityThread(); // ActivityThread 객체를 생성
            thread.attach(false);
            Looper.loop();
        }

        // 메시지 핸들러
        public void handleMessage(Message msg) {
           switch (msg.what) {
             :
             case CREATE_SERVICE:
                   handleCreateService((CreateServiceData)msg.obj);
                   break;
             case SERVICE_ARGS:
                   handleServiceArgs((ServiceArgsData)msg.obj);
                   break;
             case STOP_SERVICE:
                   handleStopService((IBinder)msg.obj);
                   break;
             :
           }
       }
}
```

Looper는 각 스레드의 메시지 루프를 실행하는 클래스.

ActivityThread 객체는 액티비티 매니저 서비스와의 상호작용을 통해 안드로이드 애플리케이션 프로세스의 메인 스레드 실행 및 액티비티 스케줄링 등을 수행한다.  

attach() 메서드는 다음 그림과 같은 과정을 보인다.  

![JS7](/images/post/JS7.png "JS7")  

⑴ ActivityThread - ActivityManagerProxy 객체의 attachApplication() 프록시 메서드를 호출  
⑵ ActivityManagerProxy 객체 - ActivityManagerNative 객체에 ATTACH_APPLICATION_TRANSACTION RPC 코드와 바인더 RPC 데이터를 전송  
⑶ ActivityManagerNative 객체 - ActivityManagerService에 포함된 attachApplication() 스텁 메서드를 호출  

코드를 바탕으로 attach() 메서드가 바인더 RPC를 통해 attachApplication()을 어떻게 호출하는지 알아보자.  

##### ⑴ ActivityThread - attachApplication() 프록시 메서드 호출  

```
private final void attach(boolean system) {
    :
    if (!system) {
        IActivityManager mgr = ActivityManagerNative.getDefault();
        try {
            mgr.attachApplication(mAppThread);
        } catch (RemoteException ex) {
        }
    }
}
```

이 메서드의 주요 기능은 ActivityThread와 액티비티 매니저 서비스 간에 IActivityManager 인터페이스 기반의 바인더 RPC를 위한 연결을 설정하는 것이다. 바인더 RPC 연결이 설정되면 ActivityThread는 ActivityManagerProxy 객체를 통해 액티비티 매니저 서비스에게 특정 작업을 요청할 수 있다.  

##### ⑵ ActivityManagerProxy 객체  

```
public void attachApplication(IApplicationThread app)
{
  Parcel data = Parcel.obtain();
  Parcel reply = Parcel.obtain();
  data.writeInterfaceToken(IActivityManager.descriptor);
  data.writeStrongBinder(app.asBinder());
  mRemote.transact(ATTACH_APPLICATION_TRANSACTION, data, reply, 0);
}
```

⑴ 에서 호출된 attachApplication() 프록시 메서드를 살펴보면, app 매개변수를 통해 전달받은 ApplicationThread에 대한 바인더 객체를 마샬링해서 ATTACH_APPLICATION_TRANSACTION RPC 코드와 바인더 RPC 데이터를 ActivityManagerNative 객체에 전달한다.  


##### ⑶ ActivityManagerNative 객체 - attachApplication() 스텁 메서드를 호출  

```
public boolean onTransact(int code, Parcel data, Parcel reply, int ?ags)
{
  switch (code) {
    :
    case ATTACH_APPLICATION_TRANSACTION: {
      IApplicationThread app = ApplicationThreadNative.asInterface(data.readStrongBinder());
      attachApplication(app);
      return true;
    }
    :
  }
}
```

ActivityThread가 보낸 ATTACH_APPLICATION_TRANSACTION RPC 코드와 바인더 RPC 데이터는 ActivityManagerNative 객체의 onTransact() 메서드를 통해 처리된다.  

지금까지의 과정을 정리하면 ActivityThread 객체는 attach() 메서드를 통해 액티비티 매니저 서비스가 자신을 제어할 수 있도록 바인더 RPC 연결을 설정한다. 연결이 설정되고 나면 액티비티 매니저 서비스의 attachApplication() 스텁 메서드가 호출된다.  


#### 액티비티 매니저 서비스 - attachApplication() 스텁 메서드 처리  

attachApplication() 스텁 메서드의 전체적인 동작 과정  

![JS8](/images/post/JS8.png "JS8")  

⑴ 액티비티 매니저 서비스 - ActivityManagerProxy 객체의 scheduleCreateService() 프록시 메서드 호출  

⑵ ActivityManagerProxy 객체 - ActivityThread의 ActivityManagerNative 객체에 SCHEDULE_CREATE_SERVICE_TRANSACTION RPC 코드와 바인더 RPC 데이터 전송  

⑶ ActivityManagerNative 객체 - ApplicationCreateService의 ApplicationThread 객체에 포함된 scheduleCreateService() 스텁 메서드 호출   

⑷ ApplicationThread 객체 - ApplicationCreateService의 ActivityThread에 메시지큐를 이용해 CREATE_SERVICE 메시지 전달  

⑸ ActivityThread 객체 - RemoteService 서비스 생성 및 서비스 생명주기에 따른 onCreate() 호출  

##### ⑴ 액티비티 매니저 서비스 - scheduleCreateService() 프록시 메서드 호출  

```
public final void attachApplication(IApplicationThread thread)
{
  int callingPid - Binder.getCallingPid();
  attachApplicationLocked(thread, callingPid);
}
```

이 메서드는 단순히 attachApplicationLocked() 메서드를 호출하는 역할을 한다.  
thread인자는 ApplicationThreadProxy 객체, callingPid 인자는 attachApplication() 스텁 메서드를 호출한 프로세스의 pid.

다음 코드는 attachApplicationLocked() 메서드의 주요 부분이다.  

```
// thread는 ApplicationThreadProxy 객체를 가리킴
pricate final boolean attachApplicationLocked(IApplicationThread thread, int pid)
{
  // 생성된 ActivityThread의 pid 값을 가지는 ProcessRecord를 얻음
  ProcessRecord app;
  app = mPidsSelfLocked.get(pid);

  // ProcessRecord와 ApplicationThreadProxy 객체를 연결함
  app.thread = thread;

  // 실행할 서비스의 ServiceRecord를 얻음
  ServiceRecord sr = null;

  for (int i=0; i<mPendingServices.size(); i++) {
    sr = mPendingServices.get(i); // 큐에 저장했던 RemoteService에 대한 ServiceRecord 객체를 얻는다.  
    mPendingServices.remove(i);
    i--;

    realStartServiceLocked(sr, app); // ProcessRecord와 ServiceRecord 값을 realStartServiceLocked() 메서드로 전달한다.  
  }

  return true;
}
```


![JS9](/images/post/JS9.png "JS9")  


```
private final void realStartServiceLocked(ServiceRecord r, ProcessRecord app)
{
  app.thread.scheduleCreateService(r, r.serviceInfo);
}
```

내부적으로 app.thread.scheduleCreateService() 메서드를 호출한다. app.thread에는 서비스 실행을 요청한 ActivityThread를 제어하기 위한 ApplicationThreadProxy 객체가 저장돼 있다.
app.thread.scheduleCreateService() 메서드는 ApplicationThreadProxy의 scheduleCreateService() 메서드를 호출한다.  


##### ⑵ ActivityManagerProxy 객체 - 바인더 RPC 데이터 전송  

```
public final void scheduleCreateService(IBinder token, ServiceInfo info)
{
  Parcel data = Parcel.obtain();
  data.writeStrongBinder(token);
  info.writeToParcel(data, 0);
  mRemote.transact(SCHEDULE_CREATE_SERVICE_TRANSACTION, data, null, IBinder.FLAG_ONEWAY);
}
```

scheduleCreateService() 프록시 메서드는 생성할 서비스(여기서는 RemoteService)에 대한 정보를 포함한 ServiceInfo 객체를 SCHEDULE_CREATE_SERVICE_TRANSACTION RPC 코드와 RPC 데이터를 통해 ApplicationThreadNative 객체에 전달한다.  


##### ⑶ ActivityManagerNative 객체 - scheduleCreateService() 스텁 메서드 호출  

```
public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
{
  switch(code) {
    :
    case SCHEDULE_CREATE_SERVICE_TRANSACTION: {
      IBinder token = data.readStrongBinder();
      serviceInfo info = ServiceInfo.CREATOR.createFromParcel(data);
      scheduleCreateService(token, info);
      return true;
    }
    :
  }

  return super.onTransact(code, data, reply, ?ags);
}
```

ApplicationThreadProxy 객체가 data 변수에 마샬링해서 전달한 ServiceRecord 객체(Binder 객체를 확장한 객체)와 ServiceInfo 객체를 언마샬링한 다음 각각 token과 info 변수에 저장한다. 그리고 이렇게 바인더 RPC로부터 수신한 데이터를 저장한 token과 info를 각각 ActivityThread의 scheduleCreateService() 스텁 메서드의 인자로 넘긴다.  


##### ⑷ ApplicationThread 객체 - ActivityThread로 CREATE_SERVICE 메시지 전달  

```
public final void scheduleCreateService(IBinder token, ServiceInfo info)
{
  CreateServiceData s = new CreateServiceData();
  s.token = token;
  s.info = info;
  queueOrSendMessage(H.CREATE_SERVICE, s);
}
```

scheduleCreateService() 스텁 메서드는 인자를 이용해 CreateServiceData라는 객체를 만든 다음 이를 ActivityThread 메시지 큐에 CREATE_SERVICE 메시지로 전달한다.  

지금까지의 과정을 살펴보면 다음과 같이 나타낼 수 있다.  

![JS10](/images/post/JS10.png "JS10")  

ApplicationThread는 액티비티 매니저 서비스의 제어 명령을 바인더 RPC로 수신하기 위한 용도로 사용되고,  
실제 액티비티 매니저 서비스로부터 요청받은 서비스를 실행하거나 생명주기를 관리하는 일은 ActivityThread가 처리하기 때문이다.  

##### ⑸ ActivityThread 객체 - 서비스 생성 및 서비스의 onCreate() 호출   

ActivityThread의 메시지 핸들러 코드 중 CREATE_SERVICE 메시지를 처리하는 주요 부분이며, 실질적인 처리는 handleCreateService() 메서드에서 이루어진다.

```
public void handleMessage(Message msg)
{
  switch (msg.what) {
    :
    case CREATE_SERVICE:
    handleCreateService((CreateServiceData)msg.obj);
    break;
    :
  }
}

private final void handleCreateService(CreateServiceData data)
{
  // 서비스 인스턴스 생성
  PackageInfo packageInfo = getPackageInfoNoCheck(data.info.applicationInfo);
  Service service = null;
  java.lang.ClassLoader cl = packageInfo.getClassLoader();
  service = (Service) cl.loadClass(data.info.name).newInstance();

  // 서비스 생명주기 시작
  service.onCreate();
}
```



### 정리

![JS11](/images/post/JS11.png "JS11")  

⑴ Controller 액티비티는 RemoteService 서비스를 실행하기 위해 startService() API를 통해 액티비티 매니저 서비스에 RemoteService 서비스 실행을 요청한다.  

⑵ 요청받은 서비스가 리모트 서비스인 경우 액티비티 매니저 서비스는 Zygote에게 서비스를 별도의 독립 프로세스로 실행시키기 위해 ActivityThread 생성을 요청한다.  

⑶ Zygote에 의해 생성된 ActivityThread는 attachApplication() 프록시 메서드를 통해 액티비티 매니저 서비스에게 자신을 등록한다. 이를 통해 액티비티 매니저 서비스는 생성도니 ActivityThread를 제어할 수 있다.  

⑷ 액티비티 매니저 서비스는 ⑴에서 요청받은 RemoteService 생성을 ActivityThread에 요청한다.  

⑸ ActivityThread는 요청했던 RemoteService 서비스의 인스턴스를 생성한 다음 이 서비스의 onCreate() 콜백 함수를 호출한다.  
