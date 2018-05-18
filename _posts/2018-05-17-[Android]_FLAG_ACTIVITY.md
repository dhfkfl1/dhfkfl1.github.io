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
이후, B에서 C를 실행할 때 FLAG_ACTIVITY_CLEAR_TASK 플래그를 설정하고 같은 순서로 실행해 보자.<br>
A가 루트 액티비티로 태스크에 존재하고, 이후 B가 쌓인다.<br>
B에서 C를 실행할 때, 설정된 플래그에 의해 태스크에 존재하던 A와 B가 사라지고 태스크엔 C만 존재하게 된다.<br>
이 플래그는 ActivityStarter.java 내에서 찾아볼 수 있었다.<br>
clearedTask(), setTaskFromIntentActivity(), startActivityUnchecked(), setTargetStackAndMoveToFrontIfNeeded() 등의 메소드에서 사용
~~~
boolean clearedTask = (mLaunchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
        == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK) && (mReuseTask != null);
if (startedActivityStackId == PINNED_STACK_ID && (result == START_TASK_TO_FRONT
        || result == START_DELIVERED_TO_TOP || clearedTask)) {
    mService.mTaskChangeNotificationController.notifyPinnedActivityRestartAttempt(
            clearedTask);
    return;
}
~~~
<br>
<h3>FLAG_ACTIVITY_CLEAR_TOP</h3>
액티비티에 FLAG_ACTIVITY_CLEAR_TOP 플래그를 설정할 경우, 이 플래그는 실행될 액티비티가 포함된 태스크 내에 동일한 액티비티가 존재할 경우 해당 액티비티부터 상위 액티비티를 모두 제거하고 실행된다.<br>
예를 들어, A->B->C->B로 실행해 태스크엔 A, B, C, B가 존재한다고 가정해보자.<br>
이후, C에서 B를 실행할 때 FLAG_ACTIVITY_CLEAR_TOP 플래그를 설정하고 같은 순서로 실행해 보자.<br>
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
액티비티에 FLAG_ACTIVITY_REORDER_TO_FRONT 플래그를 설정할 경우, 이 플래그는 태스크 내에 동일한 액티비티가 존재하면 톱으로 끌어올린다.<br>
예를 들어, A->B->C->B로 실행해 태스크엔 A, B, C, B가 존재한다고 가정해보자.<br>
이후, C에서 B를 실행할 때 FLAG_ACTIVITY_REORDER_TO_FRONT 플래그를 설정하고 같은 순서로 실행해 보자.<br>
A가 루트 액티비티로 태스크에 존재하고, 이후 B와 C가 태스크에 쌓인다. A->B->C<br>
C에서 B를 실행할 때, 설정된 플래그에 의해 태스크에 존재하는 B를 톱 액티비티로 끌어 올린다. 태스크에 A->C->B가 존재<br>
이 플래그는 ActivityStarter.java의 setTaskFromSourceRecord()에서 찾아볼 수 있었다.<br>
~~~
else if (!mAddingToTask && (mLaunchFlags & FLAG_ACTIVITY_REORDER_TO_FRONT) != 0) {
            final ActivityRecord top = sourceTask.findActivityInHistoryLocked(mStartActivity);
            if (top != null) {
                final TaskRecord task = top.getTask();
                task.moveActivityToFrontLocked(top);
                top.updateOptionsLocked(mOptions);
                ActivityStack.logStartActivity(AM_NEW_INTENT, mStartActivity, task);
                deliverNewIntent(top);
                mTargetStack.mLastPausedActivity = null;
                if (mDoResume) {
                    mSupervisor.resumeFocusedStackTopActivityLocked();
                }
                return START_DELIVERED_TO_TOP;
            }
        }
~~~

<h3>FLAG_ACTIVITY_TASK_ON_HOME</h3>
액티비티에 FLAG_ACTIVITY_TASK_ON_HOME 플래그를 설정할 경우, 이 플래그가 설정된 액티비티를 실행한 뒤, 이전 버튼을 태스크가 종료되면 이전에 사용된 앱의 태스크로 돌아가는 것이 아니라 홈 태스크로 돌아가게 한다.<br>
예를 들어, 3개의 태스크 HOME, A, B가 존재한다고 가정해보자.<br>
먼저 홈에서 A 앱을 실행한다면, HOME 태스크엔 home이란 액티비티가 존재하고
A 태스크에 A1이란 액티비티가 생성된다.<br>
A1에서 A2를 실행하면 A 태스크엔 A1->A2가 존재<br>
A2에서 B 앱의 B1을 실행하면 B 태스크가 생성되고 B1이 태스크에 적재된다.<br>
현재 화면엔 B1이 보여지고 있고, 이전 키를 누르면 A2가 보여질 것이다.<br>
태스크의 적재는 HOME->A->B로 되었기 때문이다.<br>
이제 B1에 FLAG_ACTIVITY_TASK_ON_HOME 플래그를 설정하고 같은 순서로 실행해 보자.<br>
HOME 태스크엔 home 액티비티가 존재하고 A 태스크엔 A1->A2 액티비티가 존재한다.<br>
A2에서 B1을 실행하여 B 태스크가 생성되고 FLAG_ACTIVITY_TASK_ON_HOME가 설정된 B1이 적재된다.<br>
B1에서 이전 키를 눌러 B 태스크를 사라지게 하면 HOME 태스크의 home 액티비티가 화면에 보여진다.<br>
태스크의 적재는 HOME->A->B 였지만 FLAG_ACTIVITY_TASK_ON_HOME 플래그로 인해 A->HOME->B로 바뀌었기 때문이다.<br>
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
startConfirmCredentialIntent(), updateTaskReturnToType() 등의 메소드에 사용

~~~
void startConfirmCredentialIntent(Intent intent, Bundle optionsBundle) {
    intent.addFlags(FLAG_ACTIVITY_NEW_TASK |
            FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS |
            FLAG_ACTIVITY_TASK_ON_HOME);
    ActivityOptions options = (optionsBundle != null ? new ActivityOptions(optionsBundle)
                    : ActivityOptions.makeBasic());
    options.setLaunchTaskId(mSupervisor.getHomeActivity().getTask().taskId);
    mService.mContext.startActivityAsUser(intent, options.toBundle(), UserHandle.CURRENT);
}
~~~

---
private void updateTaskReturnToType(
        TaskRecord task, int launchFlags, ActivityStack focusedStack) {
    if ((launchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME))
            == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME)) {
        // Caller wants to appear on home activity.
        task.setTaskToReturnTo(HOME_ACTIVITY_TYPE);
        return;
    } else if (focusedStack == null || focusedStack.isHomeStack()) {
        // Task will be launched over the home stack, so return home.
        task.setTaskToReturnTo(HOME_ACTIVITY_TYPE);
        return;
    } else if (focusedStack != null && focusedStack != task.getStack() &&
            focusedStack.isAssistantStack()) {
        // Task was launched over the assistant stack, so return there
        task.setTaskToReturnTo(ASSISTANT_ACTIVITY_TYPE);
        return;
    }

    // Else we are coming from an application stack so return to an application.
    task.setTaskToReturnTo(APPLICATION_ACTIVITY_TYPE);
}
---
