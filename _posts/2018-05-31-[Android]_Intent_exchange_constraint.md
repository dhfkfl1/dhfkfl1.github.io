---
layout: post
title: "[180531] Activity 간 데이터를 주고받을 때의 제약사항"
date:   2018-05-31 17:55:25 +0900
---
하나의 액티비티에서 다른 액티비티로 데이터를 주고받기 위해 인텐트를 사용한다.<br>
아래는 A 액티비티에서 B 액티비티를 실행하는 과정의 예시이다.<br>
A 액티비티에서 인텐트를 작성하고 startActivity()를 통해 액티비티 매니저로 전달한다.<br>
액티비티 매니저는 전달받은 인텐트에서 정보를 추출해 패키지 매니저로 전달한다.<br>
패키지 매니저는 설치된 앱 중 수신받은 정보의 액티비티(B)가 존재하는지 확인한다.<br>
B 액티비티가 존재할 경우, 패키지 매니저는 액티비티 매니저로 실행할 액티비티(B)의 정보를 전달한다.<br>
액티비티 매니저는 제공받은 액티비티 정보를 통해 B 액티비티를 실행한다.(인텐트 전달)<br>
호출된 B 액티비티가 실행되고 인텐트를 수신한다.
<br>
<br>
위의 예시는 startActivity()로 A 액티비티가 일방적으로 B 액티비티로 정보(인텐트)를 전달하는 과정이다.<br>
startActivityForResult()는 A 액티비티와 B 액티비티의 데이터 교환을 위한 함수로 startActivity()와의 차이라면 onActivityResult()로 결과값을 전달받는 과정이 추가된다는 것이다.<br>
<br>
이번 포스팅에서는 startActivityForResult()의 데이터 전달이 제약되는 상황에 대해서 알아보려고 한다.<br>

<h3>서로 다른 태스크에서 실행된 액티비티 결과는 받을 수 없다</h3>
<br>
startActivityForResult()로 액티비티를 실행했더라도 onActivityResult()를 통해 결과를 받을 수 없는 경우를 의미한다.<br>
다시 말해, startActivityForResult()는 같은 태스크 내에 존재하는 액티비티의 결과만을 취할 수 있다는 것이다.<br>
왜 다른 태스크에서 실행된 액티비티 결과가 전달될 수 없는걸까? <br>
그 이유는 바로 태스크 순서가 달라질 수 있기 때문이다. <br>
예를 들어, A 앱과 B 앱이 존재한다고 해보자.<br>
홈에서 A 앱의 A1 액티비티를 실행한다. (홈 태스크 -> A 태스크)<br>
A 앱에서 B 앱의 B1 액티비티를 startActivityForResult()를 통해 실행한다.
(홈 태스크[홈 액티비티] -> A 태스크[A1] -> B 태스크[B1])<br>
B 앱의 B1 액티비티가 톱 액티비티인 상황에서 이전키를 누르면 B 태스크가 종료되고 A 태스크의 A1으로 이동한다.<br>
이 흐름은 아주 자연스럽고 결과값의 전달도 아무런 이상이 없다.<br>
그러나, 태스크 순서가 바뀌었을 때를 생각해보자<br>
B1 액티비티에서 홈키를 누르면 태스크의 순서가 바뀌게 된다. (A 태스크 -> B 태스크 -> 홈 태스크)<br>
홈 화면에서 '최근 실행한 앱'으로 다시 B 태스크를 활성화한다. (A 태스크 -> 홈 태스크 -> B 태스크)<br>
현재 화면엔 B1 액티비티가 보이게 되고, 여기서 이전키를 눌르면 A 앱이 아닌 홈 화면으로 돌아가게 된다. (B의 결과값이 A가 아닌 홈으로 전달)<br>
이런 문제 때문에 안드로이드에서는 서로 다른 태스크의 액티비티 결과를 전달하지 않는다.<br>

<h3>액티비티 간 주고받는 데이터는 100KB를 넘지 말자</h3><br>
A, B 액티비티가 서로 데이터를 주고받기 위해서는 인텐트를 사용하며, 인텐트는 시스템의 액티비티 매니저를 통해 목적지 액티비티로 전달된다.<br>
따라서 액티비티가 전달한 인텐트는 가장 먼저 액티비티 매니저가 받는다.<br>
여기서 액티비티와 액티비티 매니저는 서로 다른 프로세스이기 때문에 인텐트 데이터를 전달하려면 IPC(바인더 통신)를 사용해야 하고, 이를 위해 안드로이드는 바인더 드라이버를 통해 프로세스 간에 통신한다.<br>
바인더 드라이버가 전달할 수 있는 크기는 1MB이고, 그 이상의 데이터를 전송하면 다음 그림과 같이 안드로이드 바인더에서 에러가 발생한다.<br>
1MB 이내의 인텐트 객체를 전달한다면 무조건 전달될 것이라고 생각이 들겠지만, 바인더 버퍼는 여러 프로세스가 공유해서 사용하기 때문에 늘 1MB를 보장받을 수 없다.<br>
그러므로 안드로이드에서는 100KB 이내로 사용하기를 권장하고 있다.<br>
100KB 이상의 이미지나 동영상 등은 해당 데이터의 경로만 전달하는 방식<br>
~~~
frameworks\native\libs\binder\ProcessState.cpp 에 바인더 버퍼 크기가 정의되어 있다.
#define BINDER_WM_SIZE ((1 * 1024 * 1024) - sysconf(_SC_PAGE_SIZE) * 2)
                                                //_SC_PAGE_SIZE = 4096
~~~

<h3>생명주기 onPause, onStop, onDestory에서는 setResult 함수를 사용할 수 없다</h3>
A 액티비티에서 B 액티비티의 결과값을 전달받기 위해선 B 액티비티는 반드시 setResult 함수를 통해 전달하려는 인텐트를 설정하고 종료해야 한다.<br>
액티비티 생명주기 중 종료와 관련된 함수 onPause, onStop, onDestory에서 setResult 함수를 쓰면 자연스러운 느낌이 들지만, 사용하게 되면 인텐트가 전달되지 않는다.<br>
액티비티가 종료될 때 finish 함수가 호출되는데, finish 함수는 setResult 함수를 통해 설정된 인텐트를 액티비티 매니저에 전달하는 역할을 한다.<br>
결국 setResult 함수는 finish 함수 이전에 호출되어야 인텐트가 전달되는 것이다.<br>
onPause, onStop, onDestory에 setResult 함수를 사용하면 이미 finish 함수 이후에 호출되는 함수이므로 인텐트가 전달되지 못한다.
