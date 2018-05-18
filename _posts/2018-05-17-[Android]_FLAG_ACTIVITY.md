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
<br>
<h3>FLAG_ACTIVITY_NO_HISTORY</h3>
액티비티에 FLAG_ACTIVITY_NO_HISTORY 플래그를 설정할 경우, 해당 액티비티는 태스크에 쌓이지 않는다.<br>
예를 들어, A->B->C 순으로 액티비티를 실행하여 태크스에 A, B, C가 존재한다고 가정해보자.<br>
이후, B 액티비티에 FLAG_ACTIVITY_NO_HISTORY 플래그를 설정하고 같은 순서로 실행하면 태스크엔 A와 C만이 존재하게 된다.<br>
이 플래그는 ActivityRecord.java의 isNoHistory()에서 찾아볼 수 있었다.
~~~
boolean isNoHistory() {
    return (intent.getFlags() & FLAG_ACTIVITY_NO_HISTORY) != 0
            || (info.flags & FLAG_NO_HISTORY) != 0;
}
~~~
<br>
<h3>FLAG_ACTIVITY_CLEAR_TASK</h3>
액티비티에 FLAG_ACTIVITY_CLEAR_TASK 플래그를 설정할 경우, 해당 플래그로 액티비티를 실행하면 태스크 내의 모든 액티비티가 제거되고 실행되는 액티비티만 남는다.<br>
예를 들어, A에서 B를 실행하고 B에선 C를 실행해 태스크엔 A, B, C가 존재한다고 가정해보자.<br>
이후, B에서 C를 실행할때 FLAG_ACTIVITY_CLEAR_TASK 플래그를 설정하고 같은 순서로 실행해 보자.<br>
A가 루트 액티비티로 태스크에 존재하고, 이후 B가 쌓인다.<br>
B에서 C를 실행할 때, 설정된 플래그에 의해 태스크에 존재하던 A와 B가 사라지고 태스크엔 C만 존재하게 된다.<br>
이 플래그는 ActivityStarter.java 내에서 찾아볼 수 있었다.<br>
clearedTask(), setTaskFromIntentActivity(), startActivityUnchecked(), setTargetStackAndMoveToFrontIfNeeded() 등의 메소드에서 사용
~~~
boolean clearedTask = (mLaunchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
        == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK) && (mReuseTask != null);
if (startedActivityStackId == PINNED_STACK_ID && (result == START_TASK_TO_FRONT
        || result == START_DELIVERED_TO_TOP || clearedTask)) {
    // The activity was already running in the pinned stack so it wasn't started, but either
    // brought to the front or the new intent was delivered to it since it was already in
    // front. Notify anyone interested in this piece of information.
    mService.mTaskChangeNotificationController.notifyPinnedActivityRestartAttempt(
            clearedTask);
    return;
}
~~~
<br>
<h3>FLAG_ACTIVITY_CLEAR_TOP</h3>
액티비티에 FLAG_ACTIVITY_CLEAR_TOP 플래그를 설정할 경우, 이 플래그는 실행될 액티비티가 포함된 태스크 내에 동일한 액티비티가 존재할 경우 해당 액티비티부터 상위 액티비티를 모두 제거하고 실행된다.<br>
예를 들어, A->B->C->B로 실행해 태스크엔 A, B, C, B가 존재한다고 가정해보자.<br>
이후, C에서 B를 실행할때 FLAG_ACTIVITY_CLEAR_TOP 플래그를 설정하고 같은 순서로 실행해 보자.<br>
A가 루트 액티비티로 태스크에 존재하고, 이후 B와 C가 태스크에 쌓인다. A->B->C<br>
C에서 B를 실행할 때, 설정된 플래그에 의해 태스크에는 B부터 톱 액티비티까지 모두 제거된다. 태스크엔 A만 존재<br>
기존 B, C가 사라지고 FLAG_ACTIVITY_CLEAR_TOP로 설정된 B 액티비티가 다시 실행된다. 태스크에 A->B가 존재<br>
이 플래그는 ActivityStarter.java와 ActivityManagerService.java 내에서 찾아볼 수 있었다.<br>
ActivityStarter에선 startActivityUnchecked(), setTaskFromIntentActivity(), setTaskFromSourceRecord() <br>
ActivityManagerService에선 startNextMatchingActivity(), reportAssistContextExtras() 메소드에서 사용

~~~
if ((mLaunchFlags & FLAG_ACTIVITY_CLEAR_TOP) != 0
                    || isDocumentLaunchesIntoExisting(mLaunchFlags)
                    || mLaunchSingleInstance || mLaunchSingleTask) {
                final TaskRecord task = reusedActivity.getTask();

                final ActivityRecord top = task.performClearTaskForReuseLocked(mStartActivity,
                        mLaunchFlags);


                if (reusedActivity.getTask() == null) {
                    reusedActivity.setTask(task);
                }

                if (top != null) {
                    if (top.frontOfTask) {
                        top.getTask().setIntent(mStartActivity);
                    }
                    deliverNewIntent(top);
                }
            }
~~~

<h3>FLAG_ACTIVITY_REORDER_TO_FRONT</h3>
