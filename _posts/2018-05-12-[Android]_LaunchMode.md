---
layout: post
title: <180512> Activity에서 LaunchMode란?
---

<p>ActivityRecord.java와 ActivityStart.java를 보다보면 LaunchMode라는 속성이 등장하는데<br>
이 속성이 무엇을 의미하는지 알아보았다.</p>

<h3>액티비티의 LaunchMode(실행모드)</h3>
태스크 내에는 A->B->C->A와 같이 중복된 A 액티비티가 존재할 수 있다.<br>
그러나 이런 중복을 원하지 않는 경우가 있다. 그를 위해 존재하는 것이 LaunchMode(실행모드)이다.<br>

<h3>LaunchMode의 종류</h3>
<p>액티비티의 실행모드에는 4가지 종류가 있다. <br>
<br>
1. <strong>standard</strong> 모드는 LaunchMode의 기본값으로, 태스크 내에 중복된 액티비티를 허용한다.<br>
2. <strong>singleTop</strong> 모드는 태스크의 톱 액티비티와 중복된 액티비티를 허용하지 않는다.<br>
   예를 들어, 태스크 내에 A->B->C가 존재하고 C 액티비티는 C 액티비티를 실행할 수 있다고 하면<br>
   A->B->C->C가 되는데, singleTop 모드에서는 톱 액티비티인 C와의 중복을 허용하지 않으므로 A->B->C가 된다.<br>
3. <strong>singleTask</strong> 모드는 태스크 내에 중복된 액티비티를 허용하지 않는다. 또, singleTask로 설정된 액티비티는 다른 태스크친화력을 가진 태스크에서는 절대 실행되지 않는다.<br>
4. <strong>singleInstance</strong> 모드는 태스크 내에 중복된 액티비티를 허용하지 않을 뿐만 아니라, 다른 액티비티도 허용하지 않는다.<br>
   때문에 태스크 내에는 singleInstance로 설정된 액티비티 하나만 유일하게 존재한다.</p>
