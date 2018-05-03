---
layout: post
title: <180503> Activity Start study
---

<h3>ActivityStart는 어떤 작업을 하는 클래스일까?</h3>
<p>
ActivityStart.java에는 액티비티가 시작되기 전 필요한 작업을 수행하는 다양한 메소드로 구성되어 있는 것 같다.
</p>

~~~
private void setTaskToCurrentTopOrCreateNewTask() {
        mTargetStack = computeStackFocus(mStartActivity, false, null /* bounds */, mLaunchFlags,
                mOptions);
        if (mDoResume) {
            mTargetStack.moveToFront("addingToTopTask");
        }
        final ActivityRecord prev = mTargetStack.topActivity();
        final TaskRecord task = (prev != null) ? prev.getTask() : mTargetStack.createTaskRecord(
                mSupervisor.getNextTaskIdForUserLocked(mStartActivity.userId), mStartActivity.info,
                mIntent, null, null, true, mStartActivity.mActivityType);
        addOrReparentStartingActivity(task, "setTaskToCurrentTopOrCreateNewTask");
        mTargetStack.positionChildWindowContainerAtTop(task);
        if (DEBUG_TASKS) Slog.v(TAG_TASKS, "Starting new activity " + mStartActivity
                + " in new guessed " + mStartActivity.getTask());
    }
~~~
<p>위의 코드는 Task에 대한 작업으로, Task를 스택상 맨위로 설정하거나 새로운 Task를 생성하는 메소드이다.
ActivityStart.java에서는 액티비티 관리를 위해 TaskRecord와 ActivityRecord에 대한 참조가 잦은 것 같다.</p>
