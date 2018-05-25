---
layout: post
title: "[180524] Activity에 쓰이는 FLAG(2)"
date:   2018-05-24 15:55:25 +0900
---
지난 포스팅에 이어 Activity에 쓰이는 FLAG를 정리하려고 한다.<br>

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
FLAG_ACTIVITY_BROUGHT_TO_FRONT는 시스템에서 설정하는 플래그로, 같은 태스크 내에 액티비티가 존재하고 그 액티비티의 실행모드가 singleTask이면 자동으로 설정된다고 한다.<br>
이 플래그에 대한 자세한 설명은 찾아볼 수 없었으나 이름으로 유추해 보았을 때 아래의 예시와 같은 동작을 보일 것 같다. <br>
A->B->C 액티비티들이 태스크에 존재한다고 가정하고, C 액티비티에서 singleTask 모드로 설정된 B 액티비티를 실행하려고 했을 때, B 액티비에는 FLAG_ACTIVITY_BROUGHT_TO_FRONT 플래그가 설정된다는 이야기같다.<br>
화면엔 FLAG_ACTIVITY_BROUGHT_TO_FRONT로 설정된 톱 액티비티 B가 출력될 것이다.<br>
그런데, 이 플래그가 단순히 톱 액티비티를 표현하기 위한 것으로 쓰이는 것은 이해가 가질 않는다. 뭔가 다른 기능에 사용되는지 공부를 해가며 찾아봐야 겠다.<br>
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
setTargetStackAndMoveToFrontIfNeeded() 메소드에 사용

~~~
if (topTask != null
        && (topTask != intentActivity.getTask() || topTask != focusStack.topTask())
        && !mAvoidMoveToFront) {
    mStartActivity.intent.addFlags(Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT);
~~~

<h3>FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS</h3>
FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS 플래그가 설정된 액티비티는 '최근 실행된 액티비티' 목록에 나타나지 않는다.<br>
예를 들어, A 앱을 실행해 A1 액티비티가 실행되고 A1에서 A2 액티비티를 실행시키면 태스크엔 A1->A2가 존재할 것이다.<br>
A2가 톱 액티비티인 상태에서 홈 버튼을 누르면 홈 화면으로 전환될 것이고, 최근 실행된 앱을 보면 A 앱의 A2 모습이 보일 것이다.<br>
만약 A2에 FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS 플래그가 설정된 상태로 실행됐다면
최근 실행된 앱의 목록에서 A2의 모습이 아닌 A1가 출력될 것이다.<br>
이 플래그는 ActivityStater.java와 ActivityRecord.java에서 찾아볼 수 있었다.<br>
startActivity(), startConfirmCredentialIntent() 메소드와 ActivityRecord의 생성자에 사용

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

<h3>FLAG_ACTIVITY_FORWARD_RESULT</h3>
FLAG_ACTIVITY_FORWARD_RESULT 플래그는 결과값 전달 과정에서의 번거로움을 줄이는 역할을 한다.<br>
A 액티비티에서 B 액티비티를 실행하는 방법은 startActivity(intent)와 startActivityForResult(intent)가 있다.<br>
후자의 경우, B 액티비티에서 setResult()를 정의한 뒤에 종료를 하게 되며 B를 호출했던 A는 onActivityResult()를 통해 B의 결과값을 전달받을 수 있다.<br>
액티비티가 2개인 위와 같은 상황에선 결과값 전달과정의 번거로움을 느낄 수 없을 것이다.<br>
그러나 만약 A->B->C인 상황에서 C에서 A로 결과값을 전달하려면 C에서 B로 전달하기 위해 C 액티비티에서 setResult()를 정의하고 finish()하고, B에서 다시 onActivityResult()로 C의 결과값을 가져와 A에게 전달하기 위해 setResult()를 정의하고 finish()해야 한다.<br>
FLAG_ACTIVITY_FORWARD_RESULT 플래그를 사용하면 결과값 전달 과정이 보다 효율적이게 된다. A->B->C인 상황은 동일하고, B 액티비티에서 C 액티비티를 실행할 때 FLAG_ACTIVITY_FORWARD_RESULT를 설정했다고 가정해보자<br>
이 경우 C에서 setResult()를 정의하고 finish()하면, B에서 C의 결과값을 가져오기 위한 처리를 하지 않아도 된다. B에서 finish()만 해도 A에서 onActivityResult()를 통해 C의 결과값을 받을 수 있게 된다.<br>
사용법은 A에서 B를 실행할 때 startActivityForResult()를 사용하고, B에서 C를 실행할 때 startActivity()와 FLAG_ACTIVITY_FORWARD_RESULT 플래그를 설정<br>
이 플래그는 ActivityStarter.java 내에서 찾아볼 수 있었다.<br>
startActivity() 메소드에 사용

~~~
if ((launchFlags & Intent.FLAG_ACTIVITY_FORWARD_RESULT) != 0 && sourceRecord != null) {
    // Transfer the result target from the source activity to the new
    // one being started, including any failures.
    if (requestCode >= 0) {
        ActivityOptions.abort(options);
        return ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT;
    }
    resultRecord = sourceRecord.resultTo;
    if (resultRecord != null && !resultRecord.isInStackLocked()) {
        resultRecord = null;
    }
    resultWho = sourceRecord.resultWho;
    requestCode = sourceRecord.requestCode;
    sourceRecord.resultTo = null;
    if (resultRecord != null) {
        resultRecord.removeResultsLocked(sourceRecord, resultWho, requestCode);
    }
    if (sourceRecord.launchedFromUid == callingUid) {

        callingPackage = sourceRecord.launchedFromPackage;
    }
}
~~~

<h3>FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY</h3>
FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY 플래그는 '최근 실행된 앱'을 통해 실행되었을 경우 시스템에 의해 자동적으로 설정되는 플래그라고 한다.<br>
예를 들어, A 앱을 실행해 A1->A2->A3 액티비티들을 실행한 뒤 홈 버튼을 눌러 홈 화면으로 돌아온다.<br>
다시 '최근 실행된 앱'을 통해 A 앱을 실행하면 A3 액티비티에 FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY 플래그가 자동으로 설정된다.<br>
이 플래그는 ActivityStackSupervisor.java에서 찾아볼 수 있었다.<br>
startActivityFromRecentsInner() 메소드에 사용

~~~
           intent.addFlags(Intent.FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY);
~~~


<h3>FLAG_ACTIVITY_MULTIPLE_TASK</h3>
FLAG_ACTIVITY_MULTIPLE_TASK 플래그는 단독으로 설정될 경우, 동작에 영향을 주지 않는 의미없는 플래그이다.<br>
FLAG_ACTIVITY_MULTIPLE_TASK이 동작에 영향을 주기 위해선 FLAG_ACTIVITY_NEW_TASK와 함께 사용되어야 하는데, 두 플래그를 설정하고 액티비티를 실행하게 되면 새로운 태스크가 생성되고 그곳에 해당 액티비티가 실행된다.<br>
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
setInitialState(), computeLaunchingTaskFlags(), getReusableIntentActivity(), adjustLaunchFlagsToDocumentMode(), isDocumentLaunchesIntoExisting() 등의 메소드에 사용

~~~
boolean putIntoExistingTask = ((mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0 &&
        (mLaunchFlags & FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
        || mLaunchSingleInstance || mLaunchSingleTask;
~~~

<h3>FLAG_ACTIVITY_NEW_TASK</h3>
FLAG_ACTIVITY_NEW_TASK 플래그를 사용해 액티비티를 실행하면 새로운 태스크를 생성하고 그 태스크에 액티비티가 추가된다.<br>
그러나, 실행한 액티비티와 동일한 친화력(affinity)를 가진 태스크가 있다면 새로운 태스크를 생성하지 않고 그 태스크에 추가된다.<br>
FLAG_ACTIVITY_MULTIPLE_TASK 플래그와 함께 사용될 경우, 무조건 새로운 태스크를 생성하고 그곳에 액티비티를 추가한다.<br>
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
postStartActivityProcessing(), startConfirmCredentialIntent(), startActivityUnchecked(), sendNewTaskResultRequestIfNeeded(), computeLaunchingTaskFlags() 등의 메소드에 사용

~~~
boolean clearedTask = (mLaunchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
        == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK) && (mReuseTask != null);
~~~

<h3>FLAG_ACTIVITY_NO_ANIMATION</h3>
FLAG_ACTIVITY_NO_ANIMATION 플래그는 설정하면, 액티비티를 실행할 때 사용될 수 있는 애니메이션 효과들의 적용을 막는다.<br>
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
setInitialState() 메소드에 사용
~~~
        mNoAnimation = (mLaunchFlags & FLAG_ACTIVITY_NO_ANIMATION) != 0;
~~~


<h3>FLAG_ACTIVITY_NO_USER_ACTION</h3>
FLAG_ACTIVITY_NO_USER_ACTION 플래그는 onUserLeaveHint()가 실행되는 것을 차단하는 역할을 한다.<br>
onUserLeaveHint() 메소드는 사용자의 액션없이 액티비티가 실행되거나 전환되는 경우 호출되는 메소드이다.<br>
예를 들어, A 앱을 사용하는 중에 전화가 와서 화면이 전환되는 경우 onUserLeaveHint()가 호출되는데 FLAG_ACTIVITY_NO_USER_ACTION 플래그를 설정하면 onUserLeaveHint()의 호출을 차단한다.
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
setInitialState() 메소드에 사용

~~~
        mSupervisor.mUserLeaving = (mLaunchFlags & FLAG_ACTIVITY_NO_USER_ACTION) == 0;
~~~

<h3>FLAG_ACTIVITY_SINGLE_TOP</h3>
FLAG_ACTIVITY_SINGLE_TOP 플래그는 이전 포스팅에서의 LaunchMode에서의 singleTop과 같은 역할을 한다.<br>
예를 들어, 태스크에 A->B->C 액티비티가 적재되어있고, 톱 액티비티인 C에서 다시 C 액티비티를 실행할 경우 A->B->C->C의 액티비티가 태스크에 적재될 것이다.<br>
그러나, C에서 새로운 C를 실행시킬 때 FLAG_ACTIVITY_SINGLE_TOP 플래그를 설정해 실행할 경우 톱 액티비티의 중복을 막아 A->B->C의 액티비티가 태스크에 남게 된다.
이 플래그는 ActivityStarter.java에서 찾아볼 수 있었다.<br>
startActivityUnchecked(), setTaskFromIntentActivity(), setTaskFromInTask() 등의 메소드에 사용

~~~
if (top != null && top.realActivity.equals(mStartActivity.realActivity)
        && top.userId == mStartActivity.userId) {
    if ((mLaunchFlags & FLAG_ACTIVITY_SINGLE_TOP) != 0
            || mLaunchSingleTop || mLaunchSingleTask) {
        mTargetStack.moveTaskToFrontLocked(mInTask, mNoAnimation, mOptions,
                mStartActivity.appTimeTracker, "inTaskToFront");
        if ((mStartFlags & START_FLAG_ONLY_IF_NEEDED) != 0) {
            // We don't need to start a new activity, and the client said not to do
            // anything if that is the case, so this is it!
            return START_RETURN_INTENT_TO_CALLER;
        }
        deliverNewIntent(top);
        return START_DELIVERED_TO_TOP;
    }
}
~~~
