---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager10"
description:
headline:
modified: 2019-09-25
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


## PackageManager10   


---------------------------------------

### 멘토님 의견  

① Git log를 봐도 5년간 change가 없는 곳이다.  

② 안드로이드의 경우 OpenSource이기 때문에 간혹 라이센스 문제라던지 기타 등등의 이유로 이해할 수 없는 라이브러리를 쓰기도 한다.(경험적인 의견)  

③ 어쨌든 개선 포인트에 대해서 실험 결과를 바탕으로 결론까지 도출해 보면 좋을 것 같다.  

④ Contribution의 경우 특별한 방법이 있는 것은 아니고, 지금 Git을 통해 소스를 받은 상태인데, 코드 수정 후에 Commit을 만들고 repo upload 등을 통해 업로드해봐라.  
(리뷰어 점수를 받고 merge가 되니까 올리는 것은 걱정하지 말라.)


### 100번 측정(?)  

![PM10_1](/images/post/PM10_1.png "PM10_1")  

100번을 측정하고 보니, 개선이 안되어있었다. 이유를 찾아보니, 잘못된 로그로 측정하고 있었다.  

100번 측정하는 것은 다음 주에 다시 진행할 예정이다.  


### strlcpy 수정  

로그 수정하기 위해 코드를 보다가 수정하려던 strlcpy()를 잘못 작성한 것을 확인했다.  
기존 strlcpy()는 마지막에 널값을 추가해주는데, 우리가 작성한 코드는 널값을 넣어주지 않았었다.  


-수정 전-  
```
size_t                 
strlcpy(char* dst, const char* src, size_t siz)    
{
   memcpy(dst, src, siz);

   return strlen(src);
}
```


-수정 후-  
```
size_t                 
strlcpy(char* dst, const char* src, size_t siz)    
{
   size_t srclen, returnV = strlen(src);

   siz--;

   srclen = returnV;

   if (srclen > siz)
      srclen = siz;

   memcpy(dst, src, srclen);
   dst[srclen] = '\0';

   return returnV;
}
```



### 문제점  

(추측) 말 그대로 copy기 때문에 우리가 바꾼 부분이 첫 번째 부팅과 새로 앱을 깔았을 경우에서만 사용되는 것 같음  
-> 검증해야 함, 논문을 쓸 수 있을지?(매번 켤 때마다 개선되는 것이 아니기 때문에)  


### 앞으로 방향  

1. 전체 부팅시간을 측정해보고 개선이 된다면 그대로 부팅속도 향상 쪽으로  
2. 부팅속도 개선이 안된다면 메모리 블록 복사를 통한 안드로이드 개선쪽으로(다른 곳에서도 strlcpy를 사용하기 때문에)  

기존 논문 제목과 목차를 다음과 같이 하려고 했었는데, 방향을 결정하고 PPT 제작 예정  

![PM10_2](/images/post/PM10_2.png "PM10_2")  

![PM10_3](/images/post/PM10_3.png "PM10_3")  
