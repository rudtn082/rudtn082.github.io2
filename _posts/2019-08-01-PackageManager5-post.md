---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager5"
description:
headline:
modified: 2019-08-01
category: Android
tags: [Android]
imagefeature: cover_2.jpg
mathjax:
chart:
comments: true
share: ture
featured: true
---

# [안드로이드 프레임워크 개선]  


## PackageManager5   


---------------------------------------

### scanPackageDirtyLI  

지난주 빌드를 통해 로그를 출력하는 것을 성공했기 때문에 이번에는 PackageManagerService.java의 **scanPackageDirtyLI** 메소드를 분석하기로 했다.  


### scanPackageDirtyLI 분석  

Version : android-6.0.1_r77  
PackageManagerService.java  
Line : 6482 ~ 7545  

소스 코드 : https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.1_r77/services/core/java/com/android/server/pm/PackageManagerService.java  

일단 개선이 가능할 것 같다고 생각하는 부분에 대해서 작성 해 보았다.  


#### 1. Line : 6638  

![PM5_1](/images/post/PM5_1.png "PM5_1")  

이 부분은 먼저, 6595번째 줄에서 pkg가 시스템 앱이 아닌 경우 다음과 같이 null로 지정해주었다.  

```
if (!isSystemApp(pkg)) {
    // Only system apps can use these features.
    pkg.mOriginalPackages = null;
    pkg.mRealPackage = null;
    pkg.mAdoptPermissions = null;
}
```

그러면 시스템 앱이 아닌 사용자가 설치한 애플리케이션에 대해서 for문을 이용해 처음부터 끝까지 반복하게 되는데, 사용자가 설치한 앱이 많을 수록 많은 시간이 소요될 것으로 예상되며 개선할 수 있다고도 생각한다.  


#### 2. Line : 6783    

![PM5_2](/images/post/PM5_2.png "PM5_2")  

이 부분도 마찬가지로 for문을 이용하여 0부터 N까지 반복한다. 이중 for문을 사용하기 떄문에 충분히 개선 할 수 있다고 생각한다.  


#### 3. Line : 7228  

![PM5_3](/images/post/PM5_3.png "PM5_3")  

이 부분도 마찬가지로 for문을 이용하여 0부터 N까지 반복한다. 이중 for문을 사용하기 떄문에 충분히 개선 할 수 있다고 생각한다.  


#### 그외 개선과 관련된 생각..  

scanPackageDirtyLI 메소드에서 시스템 앱일 경우와 아닌 경우를 나누는 경우가 몇 번 존재했던 것 같다. 애초에 scanPackageDirtyLI를 두 개를 만들고 매개변수를 다르게 오버로딩하여 시스템 앱일 경우와 아닌 경우 다른 메소드를 실행하는 것은 어떨까?...  
