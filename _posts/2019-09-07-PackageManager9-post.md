---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager9"
description:
headline:
modified: 2019-09-07
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


## PackageManager9   


---------------------------------------


### 문제의 if문  

지난주 아래의 if문이 무조건 true로 밖에 나올 수 없다고 생각했었는데, strlcpy()를 수정하기 전에 실제로 그러한지 확인을 했다.  

```
if (strlcpy(localFileName, nativeLibPath.c_str(), sizeof(localFileName)) != nativeLibPath.size()) {
    ALOGD("Couldn't allocate local file name for library");
    return INSTALL_FAILED_INTERNAL_ERROR;
}
```

strlcpy()의 결과로 nativeLibPath.c_str()의 문자열의 길이와 nativeLibPath.size()를 비교하게 되는 코드인데,  
결국 리턴 받은 utf_chars_의 길이와 strlen(utf_chars_)를 수행하는 꼴이 된다.  
혹시 우리가 모르는 예외가 있을 수 있다고 생각해 직접 비교하는 코드를 만들어 확인했다.  

![PM9_1](/images/post/PM9_1.png "PM9_1")  

0~30 길이의 문자열을 랜덤으로 생성해서, strlcpy의 결과와 해당 문자열을 strlen 한 결과를 비교하여 false가 나오는지 확인했다.  
결과는 10만 번을 수행했지만 다음과 같이 모두 true로 나왔다.  

![PM9_2](/images/post/PM9_2.png "PM9_2")  
