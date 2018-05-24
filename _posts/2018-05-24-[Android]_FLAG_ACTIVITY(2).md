---
layout: post
title: "[180524] Activity에 쓰이는 FLAG(2)"
date:   2018-05-24 15:55:25 +0900
---
지난 포스팅에 이어 Activity에 쓰이는 FLAG를 정리하려고 한다. '이것이 안드로이드다(박성근 저)<br>

<h3>앱으로 복귀하는 방법</h3>
실행한 앱으로 복귀하는 방법에는 '최근 실행 앱'을 통한 방법과 홈에서 해당 앱을 실행하는 방법 두 가지가 있다.<br>
두 가지 방법은 모두 동일하게 동작할 것이라고 생각했지만 차이가 있다고 한다.<br>
'최근 실행 앱'을 통한 복귀는 태스크만 상위로 변경해 실행하는 방법이라고 한다.<br>
홈 화면에서 해당 앱을 눌러 복귀하는 방식은 해당 앱의 액티비티를 startActivity 함수를 통해 직접 실행하는 방법이라고 한다.<br>


<h3>FLAG_ACTIVITY_RESET_TASK_IF_NEEDED</h3>
FLAG_ACTIVITY_RESET_TASK_IF_NEEDED 플래그는 태스크 내에 정리해야 할 액티비티가 존재한다면 정리하라고 시스템에 알려 주는 역할을 한다.<br>
정리해야 할 액티비티의 구별은 clearTaskOnLaunch와 finishOnTaskLaunch 액티비티 속성을 보고 판단한다.<br>
clearTaskOnLaunch 액티비티 속성은 액티비티가 실행될 때 기존의 태스크가 존재하는 경우 태스크를 모두 비우는 역할을 한다. 마치 FLAG_ACTIVITY_CLEAR_TASK 플래그같은 동작을 보이지만 FLAG_ACTIVITY_RESET_TASK_IF_NEEDED가 설정되어야만 동작한다는 점이 다르다.<br>
또, clearTaskOnLaunch 액티비티 속성은 첫 번째 액티비티에만 적용되어야 한다고 한다.<br>
finishOnTaskLaunch 액티비티 속성이 설정된 액티비티는 FLAG_ACTIVITY_RESET_TASK_IF_NEEDED 플래그 설정을 통해 태스크로 복귀할 때 정리할 대상 액티비티가 된다.<br>
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
setTargetStackAndMoveToFrontIfNeeded(), setTaskFromIntentActivity() 등의 메소드에 사용

~~~
// If the caller has requested that the target task be reset, then do so.
if ((mLaunchFlags & FLAG_ACTIVITY_RESET_TASK_IF_NEEDED) != 0) {
    return mTargetStack.resetTaskIfNeededLocked(intentActivity, mStartActivity);
}
~~~

<h3>FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET</h3>
FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET로 실행된 액티비티부터 다음에 실행되어 스택에 쌓이는 액티비티들까지 홈에서 FLAG_ACTIVITY_RESET_TASK_IF_NEEDED 플래그 설정을 통해 태스크로 복귀할 때 정리할 대상 액티비티가 된다.<br>
예를 들어, 홈 화면에서 앱 A를 실행해 FLAG_ACTIVITY_RESET_TASK_IF_NEEDED 플래그가 설정된 액티비티 A1이 실행되고 A1은 FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET 플래그가 설정된 액티비티 A2를 실행, A2에서 A3를 실행한다고 가정해보자<br>
현재 태스크에는 A1, A2, A3가 존재하고 톱 액티비티인 A3에서 홈 버튼을 눌러 홈 화면으로 돌아가 앱 A를 다시 실행시키면 화면엔 A1이 나타나게 된다.<br>
FLAG_ACTIVITY_RESET_TASK_IF_NEEDED 플래그에 의해 FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET로 설정된 A2와 A2 이후에 쌓인 A3가 정리 대상이 되었기 때문이다.<br>
이 플래그는 ActivityStack.java에서 찾아볼 수 있었다.<br>
finishActivityLocked(), resetTargetTaskIfNeededLocked() 등의 메소드에 사용

~~~
if (index < (activities.size() - 1)) {
    task.setFrontOfTask();
    if ((r.intent.getFlags() & Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET) != 0) {
        // If the caller asked that this activity (and all above it)
        // be cleared when the task is reset, don't lose that information,
        // but propagate it up to the next activity.
        ActivityRecord next = activities.get(index+1);
        next.intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);
    }
}
~~~

<h3>FLAG_ACTIVITY_BROUGHT_TO_FRONT</h3>

이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
setTargetStackAndMoveToFrontIfNeeded() 메소드에 사용

~~~
if (topTask != null
        && (topTask != intentActivity.getTask() || topTask != focusStack.topTask())
        && !mAvoidMoveToFront) {
    mStartActivity.intent.addFlags(Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT);
~~~
