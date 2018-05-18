---
layout: post
title: "[180517] Activity에 쓰이는 FLAG"
date:   2018-05-17 15:55:25 +0900

---
지금까지 조금씩 살펴본 ActivityStart.java ActivityRecord.java TaskRecord.java ActivityRecord.java 등 액티비티를 다루는 .java 파일내에는 많은 플래그가 사용되는 것을 볼 수 있었다. 이번 포스팅에선 태스크 내의 액티비티 제어를 위해 쓰이는 Flag를 조사해 보려고 한다.


<h3>Intent란?</h3>
안드로이드 앱을 구성하는 4가지 기본 요소(Activity, Service, Broadcast Receiver, Content Provider) 간의 작업 수행을 위한 정보를 전달하는 역할을 한다.
다시 말해, 인텐트는 이벤트(액션)와 그에 딸린 데이터로 안드로이드 컴포넌트를 실행하기 위한 메커니즘이라 말할 수 있다.
<br>

<h3>FLAG_ACTIVITY_NO_HISTORY</h3>
액티비티에 FLAG_ACTIVITY_NO_HISTORY 플래그를 사용할 경우, 해당 액티비티는 태스크에 쌓이지 않는다.<br>
예를 들어, A->B->C 순으로 액티비티를 실행하여 태크스에 A, B, C가 존재한다고 가정해보자.<br>
이후, B 액티비티에 FLAG_ACTIVITY_NO_HISTORY 플래그를 설정하고 같은 순서로 실행하면 태스크엔 A와 B만이 존재하게 된다.<br>
이 플래그는 ActivityRecord.java의 isNoHistory()에서 찾아볼 수 있었다.<br>
~~~
boolean isNoHistory() {
    return (intent.getFlags() & FLAG_ACTIVITY_NO_HISTORY) != 0
            || (info.flags & FLAG_NO_HISTORY) != 0;
}
~~~
<h3></h3>
