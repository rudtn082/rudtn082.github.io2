---
layout: post
title: "[인사이드 안드로이드] 챕터 10 - 자바 서비스 프레임워크"
description:
headline:
modified: 2019-05-23
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


## Chapter10 - 자바 서비스 프레임워크  


---------------------------------------

안드로이드 서비스 프레임워크는 자바 서비스 프레임워크와 네이티브 서비스 프레임워크로 나뉜다. 자바 서비스 프레임워크는 네이티브 서비스 프레임워크에서 제공하는 4가지 핵심 기능을 동일하게 제공하지만 시스템 내부에서 서비스가 동작하는 매커니즘이나 서비스 작성 방법에 있어서는 차이점이 있다.  


### 자바 서비스 프레임워크  

자바 서비스 프레임워크는 자바 기반의 애플리케이션 프레임워크에서 동작하는 자바 시스템 서비스를 개발할 때 이용하는 클래스의 집합이다. 자바 서비스 프레임워크는 JNI를 통해 네이티브 서비스 프레임워크를 재사용함으로써 자바 레이어의 서비스 사용자가 자바로 작성된 서비스뿐만 아니라 C++로 작성된 서비스도 이용할 수 있게 된다.  

![sf1](/images/post/sf1.png "sf1")  

자바 서비스 프레임워크는 네이티브 서비스 프레임워크와 다음과 같은 차이점이 있다.  

① 서비스 생성 : 자바 서비스 프레임워크에서 자바 서비스를 개발하는 방법은 두 가지다.  
  첫 번째 방법은 Binder 클래스를 상속받아 개발하는 것으로, 서비스를 정밀하게 제어해야 할 때 적잘한 방식으로 자바 시스템 서비스를 작성할 때도 쓰이는 방법이다.  
  두 번째 방법은 Service 클래스를 상속받아 개발하는 것인데 일반적으로 특정 작업을 주기적으로 백그라운드에서 수행하는 프로세스를 구현하는 데 사용된다.  
② 바인더 IPC 처리 : 자바 서비스 프레임워크에서는 바인더 IPC를 지원하기 위해 JNI를 통해 연결된 네이티브 서비스 프레임워크의 구성요소를 재사용한다.  


#### 자바 서비스 프레임워크의 계층별 요소  

다음 그림은 자바 서비스 프레임워크의 구성요소를 계층별로 표현한 것이다. 각 레이어별로 네이티브 서비스 프레임워크와 차이점은 첫째, 서비스 사용자의 서비스 레이어에 매니저(Manager) 클래스가 위치한다. 둘째, RPC 레이어에 AIDL 도구로 자동 생성된 스텁(Stub)과 프록시(Proxy) 클래스가 위치한다. 셋째, IPC 레이어에 위치한 구성요소가 JNI를 통해 네이티브 서비스 프레임워크의 구성요소와 연결돼 있다.  

![sf2](/images/post/sf2.png "sf2")  

**서비스 레이어**  

FooManager 클래스를 구현하는 이유는 SDK에 serviceManager 클래스가 포함되지 않았기 때문에 애플리케이션에서 serviceManager 클래스를 이용하여 시스템 서비스를 등록하거나 또는 시스템 서비스를 검색 할 수 없기 때문이다.  

![sf3](/images/post/sf3.png "sf3")  

시스템 서비스 개발자는 애플리케이션 개발자가 시스템 서비스를 이용할 수 있게 SDK에 래퍼 클래스를 포함시켜야 한다.  

**RPC 레이어**  

자바 서비스 프레임워크는 안드로이드 플랫폼에 포함된 AIDL(Android Interface Definition Language) 언어와 컴파일러를 이용해 서비스 프록시와 서비스 스텁을 자동으로 생성한다. AIDL은 안드로이드에서 프로세스 간의 IPC(InterProcess Communication)를 통해 상호작용하는 자바 기반의 코드를 작성하는 데 사용되는 인터페이스 정의 언어다.  

**IPC 레이어**  

자바 서비스 프레임워크를 이용해 개발한 서비스와 서비스 프록시가 상호작용할 때도 바인더 RPC를 이용한다. 바인더 RPC를 위해 네이티브 서비스 프레임워크에서는 BpBinder와 BBinder 클래스를 제공하지만 자바 서비스 프레임워크에서는 BinderProxy와 Binder 클래스가 이용된다.  
서비스 프록시에서 서비스에게 바인더 RPC 데이터를 전달하려면 바인더 IPC를 이용해야 하는데, 자바 서비스 프레임워크에서는 JNI를 통해 네이티브 서비스 프레임워크의 바인더 IPC를 재사용한다.  

![sf4](/images/post/sf4.png "sf4")  

#### 자바 서비스 프레임워크의 클래스별 상호작용  

자바 서비스 프레임워크는 바인더 RPC를 지원하기 위해 JNI를 통해 네이티브 서비스 프레임워크의 기능을 재사용하므로 서비스 클라이언트와 서비스 서버 내부의 구성요소 간에 수직 방향으로 이뤄지는 상호작용 역시 두 프레임워크 사이에 차이점이 있다. 먼저 자바 시스템 서비스 사용자가 위치한 서비스 클라이언트부터 살펴보면 다음 그림에서 서비스 사용자가 FooManager의 foo() 메서드를 호출하는 과정과 BinderProxy의 transact() 메서드가 JNI 네이티브 함수인 android_os_BinderProxy_transact() 로 BpBinder의 transact() 함수를 호출하는 과정이 추가돼 있음을 확인할 수 있다.  

![sf5](/images/post/sf5.png "sf5")  

자바 시스템 서비스가 위치한 서비스 서버를 살펴보면 다음 그림에서 BBinder의 transact() 함수가 JavaBBinder 네이티브 서비스 스텁을 이용해 Binder execTransact() 메서드를 호출하는 과정이 추가되었다.  

![sf6](/images/post/sf6.png "sf6")  

다음 그림은 자바 서비스를 시스템에 등록하는 과정에서 자바 서비스 프레임워크 구성요소가 상호작용 하는 과정을 보여준다. 네이티브 서비스 프레임워크와 비슷하게 서비스 등록과 사용에 관한 주체는 서비스를 제공하는 서비스 서버, 서비스를 이용하는 서비스 클라이언트, 서비스 매니저, 참여 주체 사이에 통신을 지원하는 바인더 드라이버로 구성된다.  

![sf7](/images/post/sf7.png "sf7")  

① 서비스 등록 요청(서비스) : 자바 서비스 프레임워크는 자바 서비스 매니저인 ServiceManager 클래스를 이용해서 이 과정을 처리한다. FooManager 서비스는 자신을 시스템에 등록하기 위해 ServiceManager의 addService() 메서드를 호출한다. ServiceManager 내부에는 BinderProxy가 있으며, BinderProxy는 컨텍스트 매니저를 가리키는 BpBinder와 JNI를 통해 연결돼 있다.  
② 서비스 등록(서비스 매니저) : ServiceManagerProxy 서비스 프록시는 addService() 메서드의 호출 정보를 RPC 데이터로 변환한다. 이때 바인더 RPC 데이터는 Parcel 클래스에 저장되어 BinderProxy에 전달되고, JNI를 통해 BpBinder에 전달된다. 그러고 나서 바인더 IPC를 통해 컨텍스트 매니저에 전달되어 FooService 서비스가 시스템에 등록된다.  
③ 서비스 검색 요청(서비스 사용자) : FooService 서비스를 사용하기 위해 네이티브 서비스 사용자는 BpServiceManager를 통해 서비스를 검색했지만 자바 서비스 사용자는 SDK에서 제공하는 getSystemService() 메서드를 호출해서 서비스를 검색한다.  
④ 서비스 검색(서비스 매니저) : getSystemService()는 ServiceManager의 getService() 메서드를 호출해 시스템에서 FooService 서비스를 검색한다. 만약 FooService 서비스가 검색되면 IFooService.Stub.Proxy 서비스 프록시를 참조하는 FooManager를 서비스 사용자에게 반환한다.  
⑤ foo() 서비스 프록시 메서드 호출(서비스 사용자) : 서비스 사용자는 FooManager의 foo() 메서드를 호출한다. 그러고 나면 IFooService.Stub.Proxy는 foo() 메서드 호출 정보를 RPC 데이터로 변환한 다음 BinderProxy를 통해 BpBinder에 전달한다.  
⑥ foo() 서비스 스텁 메서드 실행(서비스) : BBinder는 바인더 드라이버로부터 바인더 RPC 데이터를 전달받아 JavaBBinder을 통해서 Binder의 execTransact() 메서드를 호출한다. 그러고 나서 IFooService.Stub 서비스 스텁의 onTransact() 메서드로 RPC 데이터가 전달되고 이 데이터를 분석하여 FooService의 foo() 서비스 스텁 메서드를 호출한다.  

자바 서비스 프레임워크의 가장 중요한 특징은 JNI를 통해 네이티브 서비스프레임워크의 기능을 재사용한다는 점이다. 특히 IPC 레이어에 위치한 바인더 IPC 처리를 위해 BinderProxy와 Binder 클래스가 JNI를 통해 BpBinder와 BBinder 클래스의 기능을 재사용한다는 점이 주목할만하다.  


### 동작 메커니즘  

여기서 주목해서 살펴봐야 할 점은 각 클래스의 생성 과정과 JNI 네이티브 함수 설정 인데 이를 바탕으로 자바 서비스 프레임워크가 네이티브 서비스 프레임워크를 재사용하는 메커니즘을 이해할 수 있다.  

#### 자바 서비스 프레임워크 초기화  

app_process 프로세스가 실행되면 AndroidRuntime 클래스에서 startReg() 함수를 호출해서 JNI 네이티브 함수를 달빅 가상 머신으로 로딩한다. 이때 register_android_os_Binder() 함수를 호출해서 등록되는 JNI 네이티브 함수가 바로 자바 서비스 프레임워크와 관련이 있는 네이티브 함수들이다.  

```
int register_android_os_Binder(JNIEnv* env)
{
    if (int_register_android_os_Binder(env) < 0)
        return -1;
    if (int_register_android_os_BinderInternal(env) < 0)
        return -1;
    if (int_register_android_os_BinderProxy(env) < 0)
        return -1;
    if (int_register_android_os_Parcel(env) < 0)
        return -1;
    return 0;
}
```

위의 코드는 register_android_os_Binder() 함수의 일부로 총 네 종류의 함수를 호출하고 있다. 함수명에서 마지막 단어는 JNI 네이티브 함수를 사용하는 자바 클래스의 이름을 나타낸다.  

#### Binder  

Binder 클래스를 사용하려면 달빅 가상 머신에 Binder의 네이티브 메서드를 위한 JNI 네이티브 함수를 등록해줘야 한다. 이전의 코드 int_register_android_os_Binder() 함수가 호출되면 Binder 클래스의 일부 정보를 전역 변수인 gBinderOffsets에 저장하고, Binder 클래스의 네이티브 메서드와 JNI 네이티브 함수를 매핑한다.  

```
static int int_register_android_os_Binder(JNIEnv* env)
{
    jclass clazz;

    clazz = env->FindClass(kBinderPathName);    // 달빅 가상 머신에서 클래스를 찾아 Binder 클래스의 주요 정보 저장
    LOG_FATAL_IF(clazz == NULL, "Unable to find class android.os.Binder");

    gBinderOffsets.mClass = (jclass) env->NewGlobalRef(clazz);    // Binder 클래스 정보
    gBinderOffsets.mExecTransact    // execTransact() 메서드 ID
        = env->GetMethodID(clazz, "execTransact", "(IIII)Z");
    assert(gBinderOffsets.mExecTransact);

    gBinderOffsets.mObject    // mOject 필드 ID
        = env->GetFieldID(clazz, "mObject", "I");
    assert(gBinderOffsets.mObject);

    return AndroidRuntime::registerNativeMethods(   // Binder의 네이티브 메서드와 매핑되는 JNI 네이티브 함수를 달빅 가상 머신에 등록
        env, kBinderPathName,
        gBinderMethods, NELEM(gBinderMethods));
}
```


**Binder 객체 생성**  

Binder 클래스는 바인더 IPC를 위해 BBinder의 기능을 사용하기 때문에 Binder 객체가 생성될 때 BBinder가 함께 생성돼야 한다. 다음 코드는 Binder 생성자의 소스 코드다. Binder는 생성자에서 init() 네이티브 메서드를 호출한다.  

```
public class Binder implements IBinder {
    Private native final void init();
    public Binder() {
      init();
    }
}
```

다음 코드에서는 JavaBBinderHolder 클래스의 객체를 생성한 후 SetIntField() JNI 함수를 이용해 Binder의 mObject 변수에 생성된 JavaBBinderHolder 인스턴스의 주소를 저장한다.  

```
static void android_os_Binder_init(JNIEnv* env, jobject obj)
{
    JavaBBinderHolder* jbh = new JavaBBinderHolder();
    if (jbh == NULL) {
        jniThrowException(env, "java/lang/OutOfMemoryError", NULL);
        return;
    }
    ALOGV("Java Binder %p: acquiring first ref on holder %p", obj, jbh);
    jbh->incStrong((void*)android_os_Binder_init);
    env->SetIntField(obj, gBinderOffsets.mObject, (int)jbh);
}
```


**JavaBBinder 객체 생성**  

Binder 클래스의 객체가 생성될 때 BBinder 클래스의 객체가 함께 생성돼야 하는데도 Binder 객체 생성 과정에서도 BBinder 객체가 생성되는 곳은 보이지 않는다. 대신 JavaBBinderHolder 클래스의 객체를 생성하는 코드는 확인할 수 있다.  
JavaBBinderHolder 클래스의 생성자는 단순히 mObject 변수에 Binder 객체의 주소값을 저장할 뿐이다.  

실제로 JavaBBinder의 인스턴스는 JavaBBinderHolder의 get() 함수에서 생성된다. 다음 코드를 보면 get() 함수에서 직접 JavaBBinder 클래스의 인스턴스를 생성하는 것을 확인할 수 있다. JavaBBinder는 BBinder를 상속받아 구현한 클래스이므로 JavaBBinder 객체를 생성하면 BBinder의 객체도 생성되는 것으로 볼 수 있다.  

```
sp<JavaBBinder> JavaBBinderHolder::get(JNIEnv* env)
{
    AutoMutex _l(mLock);
    sp<JavaBBinder> b = mBinder.promote();
    if (b == NULL) {
        b = new JavaBBinder(env, mObject);
        mBinder = b;
    }

    return b;
}
```

다음 그림은 Binder 객체와 JavaBBinder 객체가 생성되는 과정을 요약해서 나타낸 것이다.  

![sf8](/images/post/sf8.png "sf8")  


**Binder 클래스와 JavaBBinder 서비스 스텁 클래스의 상호 작용**  

BBinder의 transact()가 호출되면 기본적으로 onTransact()가 호출된다. BBinder에서 기본으로 제공하는 바인더 RPC 함수 이외에 새로운 기능을 제공하려면 BBinder를 상속받은 서비스 스텁 클래스에서 onTransact() 메서드를 재정의해야 한다. BBinder 클래스를 상속한 JavaBBinder 서비스 스텁 클래스는 onTransact() 함수에서 Binder의 execTransact() 메서드를 호출한다.  

다음 코드는 JavaBBinder의 onTransact() 함수다. CallBooleanMethod() JNI 함수를 호출해서 Binder의 execTransact() 메서드를 호출한다.  

```
virtual status_t JavaBBinder::onTransact(
       uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags = 0)
   {
       JNIEnv* env = javavm_to_jnienv(mVM);
       :
       jboolean res = env->CallBooleanMethod(mObject, gBinderOffsets.mExecTransact,
           code, reinterpret_cast<jlong>(&data), reinterpret_cast<jlong>(reply), flags);
       :
       return res != JNI_FALSE ? NO_ERROR : UNKNOWN_TRANSACTION;
  }
```

Binder 클래스의 execTransact() 메서드가 호출되면 onTransact() 메서드를 호출한다.  

```
private boolean execTransact(int code, int dataObj, int replyObj,
        int flags) {
    Parcel data = Parcel.obtain(dataObj);
    Parcel reply = Parcel.obtain(replyObj);

    boolean res;
    try {
        res = onTransact(code, data, reply, flags);
    } catch (RemoteException e) {
        reply.setDataPosition(0);
        reply.writeException(e);
        res = true;
    } catch (RuntimeException e) {
        reply.setDataPosition(0);
        reply.writeException(e);
        res = true;
    } catch (OutOfMemoryError e) {
        RuntimeException re = new RuntimeException("Out of memory", e);
        reply.setDataPosition(0);
        reply.writeException(re);
        res = true;
    }
    reply.recycle();
    data.recycle();
    return res;
}
```

#### BinderProxy  

**BinderProxy 클래스를 위한 JNI 설정**  

BinderProxy 클래스를 사용하려면 달빅 가상 머신에 BinderProxy의 네이티브 메서드를 위한 JNI 네이티브 함수를 먼저 등록해줘야 한다.  

**BinderProxy 객체 생성**  

BinderProxy 클래스도 바인더 IPC를 수행하는데 네이티브 서비스 프레임워크의 BpBinder의 기능을 사용하므로 BinderProxy 객체가 생성될 때 BpBinder 객체가 필요하다. 따라서 BinderProxy 객체도 Parcel의 readStrongBinder() 메서드를 호출할 때 생성된다.  

![sf9](/images/post/sf9.png "sf9")  

javaObjectForIBinder() 함수를 살펴보면 NewObject() JNI 함수를 이용해 BinderProxy 객체를 생성한 다음 BinderProxy의 mObject 변수에 BpBinder 객체를 저장한다.  

```
jobject javaObjectForIBinder(JNIEnv* env, const sp<IBinder>& val)
{
    if (val == NULL) return NULL;
    if (val->checkSubclass(&gBinderOffsets)) {
        jobject object = static_cast<JavaBBinder*>(val.get())->object();
        LOGDEATH("objectForBinder %p: it's our own %p!\n", val.get(), object);
        return object;
    }
    :
    object = env->NewObject(gBinderProxyOffsets.mClass, gBinderProxyOffsets.mConstructor);

    if (object != NULL) {
        env->SetIntField(object, gBinderProxyOffsets.mObject, (int)val.get());
    }

    return object;
}
```

**BinderProxy 클래스와 BpBinder 클래스의 상호작용**  

android_os_BinderProxy_transact() 함수를 살펴보면 BinderProxy의 mObject 변수가 참조하고 있는 BpBidner 객체의 주소를 획득하여 transact() 함수를 호출한다.  

```
static jboolean android_os_BinderProxy_transact(JNIEnv* env, jobject obj,
        jint code, jobject dataObj, jobject replyObj, jint flags) // throws RemoteException
{
    Parcel* data = parcelForJavaObject(env, dataObj);
    Parcel* reply = parcelForJavaObject(env, replyObj);
    IBinder* target = (IBinder*)
        env->GetIntField(obj, gBinderProxyOffsets.mObject);

    status_t err = target->transact(code, *data, reply, flags);
}
```

#### Parcel  

Parcel 클래스는 바인더 IPC가 진행되는 동안 송신측에서 수신측으로 전달되는 데이터를 저장하는 데 사용한다. 특히, Parcel은 내부 버퍼 안에 IBinder 객체 레퍼런스를 가지고 있어 프로세스를 가로질러 이동할 때도 레퍼런스 값을 유지해야 한다. 따라서 이런 기능을 자바 서비스 프레임워크에서도 제공하기 위해 JNI를 통해 Parcel 클래스(C++)의 기능을 재사용한다.  

**Parcel 클래스의 JNI 설정**  

Parcel 클래스의 네이티브 메서드는 JNI 함수를 통해 Parcel(C++) 클래스에 포함된 이름이 동일한 멤버 함수를 대부분 호출한다.  

**Parcel 객체 생성**  

Parcel 객체를 생성하는 과정은 Binder, BinderProxy 클래스와는 조금 다르다. Parcel의 생성자를 보면 private로 선언돼 있어 인스턴스를 획득하려면 Parcel의 obtain() 메서드를 사용해야 한다.  

obtain() 메서드에서는 Parcel의 생성자가 호출된다. 생성자 내부에서는 init() 네이티브 메서드가 호출되어 JNI로 매핑된 android_os_Parcel_init() 함수가 실행되면 다음 코드에서 Parcel(C++) 클래스의 인스턴스를 생성하는 것을 확인할 수 있다.  

```
static void android_os_Parcel_init(JNIEnv* env, jobject clazz, jint parcelInt)
{
    Parcel* parcel = (Parcel*)parcelInt;
    int own = 0;
    if (!parcel) {
        own = 1;
        parcel = new Parcel;
    }

    env->SetIntField(clazz, gParcelOffsets.mOwnObject, own);
    env->SetIntField(clazz, gParcelOffsets.mObject, (int)parcel);
}
```

다음 그림은 Parcel 객체가 생성되는 과정을 그림으로 정리한 것이다.  

![sf10](/images/post/sf10.png "sf10")  

**Parcel 클래스(Java)와 Parcel 클래스(C++) 간의 상호작용**  

일반적으로 Parcel 클래스는 서비스 프록시에서 바인더 RPC 데이터를 저장할 때 사용된다. 다음 그림은 BinderProxy의 transact() 메서드가 호출된 후 Parcel 객체의 이동경로를 보여준다. 서비스 프록시에서 바인더 RPC를 진행하면 달빅 가상 머신에서 생성된 Parcel 객체가 서비스에게 전달돼야 하는데 그러기 위해서는 Parcel(Java) 객체를 Parcel(C++) 객체로 변환해야 한다. 이를 위해 자바 서비스 프레임워크에서는 parcelForJavaObject()라는 함수를 제공한다.  

![sf11](/images/post/sf11.png "sf11")  

반대로 BBinder transact() 함수를 통해 Parcel(c++) 객체를 전달받으면 Parcel(C++) 객체를 Parcel(Java) 객체로 변환해야 한다.  


### 자바 시스템 서비스 구현  

개발자가 안드로이드 플랫폼에서 동작하는 자바 시스템 서비스를 개발하려면 기존 자바 시스템 서비스의 프로그램 구조를 파악하는 것이 가장 효과적이다. 여기서는 자바 시스템 서비스 가운데 알람 매니저 서비스를 토대로 시스템 서비스의 구조를 파악해본다.  

![sf12](/images/post/sf12.png "sf12")  

#### 알람 매니저 서비스의 구조 분석  

AlarmManagerService 클래스의 계층 구조를 클래스 다이어그램으로 나타낸 것이다. 먼저 상단에는 자바 서비스 프레임워크의 구성요소인 IInterface 인터페이스와 Binder 클래스가 위치하며, IAlarmManager 서비스 인터페이스와 서비스 스텁에서 이 클래스를 상속하고 있다. 다음으로 AIDL(Android Interface Definition Language)을 통해 자동으로 생성도니 서비스 스텁 클래스와 서비스 프록시 클래스가 위치한다. 마지막으로 그림 하단에는 실질적인 알람 매니저 서비스를 구현하고 있는 AlarmManagerService 클래스와 AlarmManager 래퍼 클래스가 있다.  

**알람 매니저 서비스 구현 방식**  

알람 매니저 서비스는 AIDL을 이용해 해당 클래스를 자동으로 생성한다. IAlarmManager 인터페이스 내부에는 총 5개의 메서드가 선언돼 있다. 다음의 코드를 AIDL 컴파일러로 컴파일하면 알람 매니저 서비스의 서비스 인터페이스, 서비스 프록시, 서비스 스텁 클래스가 자동으로 생성된다.  

```
interface IAlarmManager {
    void set(int type, long triggerAtTime, in PendingIntent operation);
    void setRepeating(int type, long triggerAtTime, long interval, in PendingIntent operation);
    void setInexactRepeating(int type, long triggerAtTime, long interval, in PendingIntent operation);
    void setTime(long millis);
    void setTimeZone(String zone);
    void remove(in PendingIntent operation);
}
```

AIDL로부터 자동 생성된 서비스 스텁 클래스는 Binder 클래스의 onTransact() 메서드를 재정의하여 시스템의 바인더 RPC 기능을 추가하므로 알람 매니저 서비스의 서비스 스텁 클래스도 onTransact() 메서드를 재정의하여 IAlarmManager 인터페이스에 정의된 5개의 메서드 관련 코드를 추가했다.  

**알람 매니저 서비스 사용**  

애플리케이션 개발자가 시스템 서비스를 사용하려면 SDK의 getSystemService() 메서드를 이용해야 한다. 알람 매니저 서비스는 Context 클래스의 getSystemService() 메서드를 호출하면 애플리케이션에서 사용할 수 있으며, ApplicationContext 클래스의 getAlarmManager() 메서드에서 이를 구현하고 있다.  

```
private AlarmManager getAlarmManager() {
    synchronized (sSync) {
        if (sAlarmManager == null) {
            IBinder b = ServiceManager.getService(ALARM_SERVICE); // BinderProxy 객체를 반환받는다.
            IAlarmManager service = IAlarmManager.Stub.asInterface(b); //서비스 프록시 클래스의 객체를 획득한다.
            sAlarmManager = new AlarmManager(service); // 인스턴스를 생성하여 반환한다.
        }
    }
    return sAlarmManager;
}
```

#### HelloWorldService 시스템 서비스의 구현  

**HelloWorldService 설계**  

AIDL을 이용해 HelloWorldService 서비스 인터페이스, 서비스 프록시, 서비스 스텁 클래스를 자동으로 생성한 후 HelloWorldService와 HelloWorldManager를 구현해보자.  

**HelloWorldService 구현**  

AIDL을 이용해 IHelloWorld.aidl 소스 파일을 작성한다.  

```
package android.app;

interface IHelloWorld{
  printHello();
}
```

다음으로 안드로이드 플랫폼을 빌드할 때 IHelloWorld.aidl 파일이 AIDL 컴파일러에 의해 컴파일 될 수 있게 Android.mk 설정 파일을 수정한다.  
LOCAL_SRC_FILES 항목에서 IHelloWorld.aidl을 추가하면 된다.  

**HelloWorldService 서비스의 구현**  

서비스는 서비스 스텁 클래스를 상속받아 구현하면 된다.  

```
package android.app;

import android.os.RemoteException;
import android.content.Context;
import android.util.Log;

public class HelloWorldService extends IHelloWorld.Stub {

  private static final String TAG = "HelloWorldService";
  Context mContext;

  public HelloWorldService(Context context){

    mContext = context;
  }

  public void printHello() throws RemoteException {
    Log.i(TAG, "Hello, World!");
  }
}
```

**HelloWorldService 서비스의 등록**  

자바 시스템 서비스는 ServerThread 클래스의 run() 메서드에서 생성하여 안드로이드 플랫폼에 등록한다. ServerThread의 run() 메서드에서 HelloWorldService의 인스턴스를 생성하고 자바 서비스 매니저의 addService() 메서드를 통해 시스템에 드록한다.  


#### HelloWorldService 시스템 서비스의 이용  

이전까지 서비스를 구현해서 시스템에 등록했다. HelloWorldService 시스템 서비스를 애플리케이션 개발자들이 이용할 수 있게끔 HelloWorldManager 클래스를 구현해보자.  

**HelloWorldManager 구현**  

```
package android.app;

import android.os.RemoteException;

public class HelloWorldManager
{
  private final IHelloWorld mService;

  HelloWorldManager(IHelloWorld service) {    // IHelloWorld.Stub.Proxy 클래스의 인스턴스를 인자로 전달받는다.
    mService = service;
  }

  public void printHello() {
    try {   // IHelloWorld.Stub.Proxy 객체의 printHello() 메서드가 호출된다.
      mService.printHello();
    } catch(RemoteException) {

    }
  }
}
```

**HelloWorldManager 획득**  

애플리케이션 개발자는 getSystemService() 메서드를 통해 시스템 서비스를 사용할 수 있다.  

다음 그림은 HelloWorldService 서비스에서 HelloWorldManager 객체를 획득하는 과정을 정리한 그림이다.  

![sf13](/images/post/sf13.png "sf13")  

① 서비스 사용자는 HelloWorldService 서비스를 이용하기 위해 getSystemService() 메서드를 호출하면 getSystemService() 메서드 내부에서 getHelloWorldManager() 메서드를 호출한다.  
② getHelloWorldManager() 메서드는 서비스 매니저에게 HelloWorldService 서비스의 검색을 요청한다. 검색에 성공하면 HelloWorldService 서비스를 가리키는 BinderProxy 객체를 반환한다.  
③ getHelloWorldManager() 메서드는 반환받은 BinderProxy 객체를 HelloWorld.Stub 서비스 스텁의 asInterface() 메서드에 넘겨줘 IHelloWorld.Stub.Proxy 서비스 프록시 객체를 생성한다.  
④ getHelloWorldManager() 메서드는 생성한 IHelloWorld.Stub.Proxy 서비스 프록시 객체를 HelloWorldManager 생성자의 인자로 넘겨줘 객체를 생성한다. HelloWorldManager는 mService 변수에 넘겨 받은 IHelloWorld.Stub.Proxy 서비스 프록시 객체를 저장한다.  
⑤ 최종적으로 HelloWorldService 서비스를 사용하기 위해 getSystemService() 메서드를 호출한 곳에서는 HelloWorldManager의 객체를 반환받ㄷ는다. 애플리케이션은 HelloWorldService 서비스를 사용하기 위해 HelloWorldManager의 메서드를 호출하면 IHelloWorld.Stub.Proxy 서비스 프록시의 메서드가 호출되어 HelloWorldService 서비스와 바인더 RPC를 사용하여 상호작용한다.  


#### HelloWorldService 시스템 서비스 빌드  

```
make
```

명령어를 통해 안드로이드 플랫폼을 컴파일 한다.  
