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
1. Standard</p>
