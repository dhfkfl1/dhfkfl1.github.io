---
layout: post
title: "[180703] 안드로이드 어플리케이션의 컴포넌트와 인텐트"
date:   2018-07-03 17:55:25 +0900
---

<h3>안드로이드 어플리케이션의 구성요소</h3>
<br>
<br>
<img src="/assets/images/compo.png" width="750" height="250">
<br>
1. 액티비티(Activity)<br>
안드로이드 어플리케이션이 실행되는 기본적인 단위로서 사용자와 상호작용하는 작업을 수행하는 컴포넌트<br>

2. 서비스(Service)<br>
사용자와 직접 상호작용을 하는 액티비티와 달리 화면에 표시되지 않고 백그라운드에서 작업을 수행하는 컴포넌트<br>

3. 브로드캐스트 리시버(Broadcast Receiver)<br>
불특정 다수를 대상으로 발송된 메시지에 대해 응답할 수 있는 컴포넌트
<br>
ex> 시스템 상태와 관련된 메시지 - 배터리 부족, 언어 변경, 파일 다운로드 완료 등<br>

4. 컨텐트 프로바이더(Content Provider)<br>
어플리케이션 내의 데이터를 다른 어플리케이션과 공유할 수 있게 하는 컴포넌트<br>

5. 인텐트(Intent)<br>
안드로이드 어플리케이션을 구성하는 컴포넌트 간의 작업요청 및 데이터를 전달하는 메시지

<h3>인텐트의 생성 주체</h3>
- 안드로이드 시스템<br>
- 안드로이드 프레임워크<br>
- 어플리케이션<br>

<h3>인텐트 전송메소드</h3>
<br>
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
<br>
Intent 객체는 컴포넌트이름 / 액션 / 카테고리 / 데이터(URI) / 데이터 타입 / 추가정보(extra)로 구성됨<br>

1. 컴포넌트 이름<br>
클래스 이름 또는 manifest 파일에서 설정된 패키지 이름과 클래스 이름의 조합<br>
- 컴포넌트 이름 설정 : setComponent(), setClass(), setClassName()<br>
- 컴포넌트 이름 읽기 : getComponent()<br>
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
