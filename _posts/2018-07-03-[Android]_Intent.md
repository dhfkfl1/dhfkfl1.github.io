---
layout: post
title: "[180703] 안드로이드 애플리케이션의 컴포넌트와 인텐트"
date:   2018-07-03 17:55:25 +0900
---

<h3>안드로이드 애플리케이션의 구성요소</h3>
<br>
<br>
<img src="/assets/images/compo.png" width="750" height="250">
<br>
1. 액티비티(Activity)<br>
안드로이드 애플리케이션이 실행되는 기본적인 단위로서 사용자와 상호작용하는 작업을 수행하는 컴포넌트<br>

2. 서비스(Service)<br>
사용자와 직접 상호작용을 하는 액티비티와 달리 화면에 표시되지 않고 백그라운드에서 작업을 수행하는 컴포넌트<br>

3. 브로드캐스트 리시버(Broadcast Receiver)<br>
불특정 다수를 대상으로 발송된 메시지에 대해 응답할 수 있는 컴포넌트
<br>
ex> 시스템 상태와 관련된 메시지 - 배터리 부족, 언어 변경, 파일 다운로드 완료 등<br>

4. 컨텐트 프로바이더(Content Provider)<br>
어플리케이션 내의 데이터를 다른 애플리케이션과 공유할 수 있게 하는 컴포넌트<br>

5. 인텐트(Intent)<br>
안드로이드 애플리케이션을 구성하는 컴포넌트 간의 작업요청 및 데이터를 전달하는 메시지

<h3>인텐트의 생성 주체</h3>
- 안드로이드 시스템<br>
- 안드로이드 프레임워크<br>
- 애플리케이션<br>

<h3>인텐트 전송메소드</h3>
컴포넌트 별로 별도의 메소드가 존재<br>
1. 액티비티(Activity)<br>
- Context.startActivity()<br>
- Activity.startActivityForResult()<br>

2. 서비스(Service)<br>
- Context.startService()<br>
- Context.bindService()<br>

3. 브로드캐스트 리시버(Broadcast Receiver)<br>
- Context.sendBroadcast()<br>
- Context.sendOrderedBroadcast()<br>
- Context.sendStickyBroadcast()<br>

4. 컨텐트 프로바이더(Content Provider)<br>
위의 1 ~ 3의 컴포넌트들은 Intent 객체를 생성하고 객체 내에 메시지를 담아 전달하는 방식 <br>
그와 달리, 컨텐트 프로바이더는 ContentResolver 객체를 통해 데이터 전달<br>

<h3>인텐트 객체의 구성요소</h3>
Intent 객체는 컴포넌트이름 / 액션 / 카테고리 / 데이터(URI) / 데이터 타입 / 추가정보(extra)로 구성됨<br>

1. 컴포넌트 이름<br>
클래스 이름 또는 manifest 파일에서 설정된 패키지 이름과 클래스 이름의 조합<br>
- 컴포넌트 이름 설정 : setComponent(), setClass(), setClassName()<br>
- 컴포넌트 이름 읽기 : getComponent()<br>
<br>
~~~
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setClassName(this, Activity.class.getName());
startActivity(intent);
~~~

2. 액션(Action)<br>
수행되어야 하는 작업 내용으로 액티비티 액션과 브로드캐스트 액션으로 나뉨<br>
- 액션 설정 : setAction()<br>
- 액션 읽기 : getAction()<br>
<br>
- 액티비티 액션
  ACTION_MAIN : 시작하는 액티비티를 지정<br>
  ACTION_VIEW : 인텐트에 첨부되는 데이터의 URI가 가르키는 데이터를 사용자에게 표시<br>
  ACTION_EDIT : 인텐트에 첨부되는 데이터의 URI가 가르키는 데이터를 변경<br>
  ACTION_DELETE : 인텐트에 첨부되는 데이터의 URI가 가르키는 데이터를 삭제<br>
  ACTION_DEFAULT : VIEW와 동일한 내용<br>
  ACTION_PICK : 데이터에서 하나의 URI를 선택하여 정보를 반환<br>
  ACTION_GET_CONTENT : 데이터에서 하나의 CONTENT를 선택하여 반환<br>
  ACTION_RUN : 데이터를 실행시키는 액션<br>
  ACTION_INSERT : 빈 아이템을 작성하라는 액션<br>
  ACTION_CALL : 전화 연결을 요청하는 액션 (실제로 전달된 전화번호로 전화를 검)<br>
  ACTION_DIAL : 전화 연결을 요청하는 액션 (전화 다이얼패드 화면으로 이동)<br>
  ACTION_SENDTO : 데이터의 메시지를 보내라는 액션<br>
  ACTION_ANSWER : 전화 착신에 관한 액션<br>
  ACTION_SYNC : 모바일 디바이스의 데이터와 서버의 데이터를 동기화하라는 액션<br>
<br>
 - 브로드캐스트 액션
  ACTION_BATTERY_CHANGED : 배터리 잔량의 변화를 알림<br>
  ACTION_BATTERY_LOW : 배터리 부족 경고<br>
  ACTION_BOOT_COMPLETED : 시스템 부팅 완료 알림<br>
  ACTION_DATE_CHANGE : 날짜 변경 알림<br>
  ACTION_HEADSET_PLUG : 헤드셋이 기기에 연결 또는 분리되었음을 알림<br>
  ACTION_PACKAGE_ADDED : 어플리케이션 패키지 추가 알림<br>
  ACTION_PACKAGE_REMOVED : 어플리케이션 패키지 제거 알림<br>
  ACTION_SCREEN_ON : 스크린이 켜졌음을 알림<br>
  ACTION_TIMEZONE_CHANGED : 타임존 변경시 알림<br>
  ACTION_TIME_CHANGED : 시간 지정시 알림<br>
  ACTION_TIME_TICK : 매 시간 변경시 알림<br>
<br>
- 카테고리(Category)<br>
인텐트를 수신하여야 할 컴포넌트의 종류에 대한 정보를 포함하는 문자열을 정의<br>
  CATEGORY_BROWSABLE : 타겟 액티비티는 링크에 의해 참조되는 데이터를 보여주기 위해 호출되는 브라우저<br>
  CATEGORY_HOME : 홈화면을 보여주는 액티비티가 되어야 한다.<br>
  CATEGORY_LAUNCHER : 액티비티는 하나의 태스크에서 최초로 생성된 액티비티가 되며, 최상위 계층의 어플리케이션을 시작된다.<br>
<br>
3. 데이터(URI)<br>
호출 대상 액티비티가 처리해 주었으면 하는 데이터 주소(실제 데이터 X)<br>
ex> 액션이 'ACTION_DAIL'인 경우, 데이터 필드는 전화번호를 가진 'tel: URI'을 가져야 한다.<br>
  - URI 설정 : setData()<br>
  - URI 읽기 : getData()<br>
  - MIME 설정 : setType()<br>
  - MIME 읽기 : getType()<br>
  - URI와 MIME 동시에 설정 : setDataAndType()<br>

4. 데이터 타입<br>
인텐트 객체 내의 데이터 필드는 데이터의 '주소'만을 나타낼 뿐, 주소 자체의 해당 URI가 가르키는 데이터의 종류가 어떤 것인지 알 수가 없다.<br>
때문에, 데이터 타입 필드는 데이터의 '타입'을 지정하여 인텐트 해석 과정에서 정확하게 대상 컴포넌트를 찾을 수 있도록 한다.<br>

5. 추가 정보(Extra)<br>
인텐트의 엑스트라 필드에 필요한 값을 설정하면 안드로이드는 엑스트라 값을 Bundle 객체로 변환한다.<br>
Bundle 객체는 인텐트를 사용하여 컴포넌트를 호출할 때 onCreate() 콜백 메서드의 파라미터로 전달된다.<br>
ex> 액션이 'ACTION_TIMEZONE_CHANGED'인 경우, 새로운 시간대를 나타내는 'time-zone' 엑스트라를 제공해야 한다.<br>
액션이 'ACTION_HEADSET_PLUG'인 경우, 헤드셋 타입에 대한 'name'과 연결 상태 'state' 엑스트라를 제공해야 한다.<br>
  - 엑스트라 설정 : putExtra()<br>
  - 엑스트라 읽기 : get<String, Int, Boolean ...>Extra()<br>

6. 플래그(Flag)<br>
안드로이드 시스템에게 액티비티를 시작시키는 방법과 시작 이후 액티비티를 다루는 방법을 표시<br>
 * <a href="https://dhfkfl1.github.io/dhfkfl1.github.io/2018/05/17/Android-_FLAG_ACTIVITY.html">플래그 종류</a>
