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
