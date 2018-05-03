---
layout: post
title: <180503> Activity Start study
---

<h3>ActivityStart.java
</h3>
<p>ActivityStart.java에는 액티비티가 시작되기 전 필요한 작업을 수행하는 다양한 메소드로 구성되어 있는 것 같다.</p>

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
