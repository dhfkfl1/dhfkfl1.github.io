---
layout: post
title: "<180503> Activity Start study"
date:   2018-05-03 17:55:25 +0900
categories: jekyll update
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
<p>위의 코드는 Task에 대한 작업으로, Task를 스택상 맨위로 설정하거나 새로운 Task를 생성하는 메소드로 추측된다.<br>
setTaskToCurrentTopOrCreateNewTask()외에도  액티비티 관리를 위해 TaskRecord와 ActivityRecord에 대한 참조가 잦은 것 같다.<br>
그래서 TaskRecord와 ActivityRecord는 어떤 모습을 취하고 있는지 의문이 들었고 TaskRecord.java와 ActivityRecord.java를 들여다 보았다.</p>

<h3>TaskRecord.java내에서 TaskRecord의 형태는?</h3>
<p>액티비티 관리를 위해 TaskRecord에 대한 참조가 잦은데, 과연 TaskRecord는 무엇으로 구성되어 있을까?<br>
일단 Task를 구성하는 액티비티와 인텐트에 대한 조작을 할 수 있는 변수가 있을 것 같다. </p>

~~~
TaskRecord(ActivityManagerService service, int _taskId, ActivityInfo info, Intent _intent,
        IVoiceInteractionSession _voiceSession, IVoiceInteractor _voiceInteractor, int type) {
}

TaskRecord(ActivityManagerService service, int _taskId, ActivityInfo info, Intent _intent,
        TaskDescription _taskDescription, TaskThumbnailInfo thumbnailInfo) {
}

private TaskRecord(ActivityManagerService service, int _taskId, Intent _intent,
        Intent _affinityIntent, String _affinity, String _rootAffinity,
        ComponentName _realActivity, ComponentName _origActivity, boolean _rootWasReset,
        boolean _autoRemoveRecents, boolean _askedCompatMode, int _taskType, int _userId,
        int _effectiveUid, String _lastDescription, ArrayList<ActivityRecord> activities,
        long _firstActiveTime, long _lastActiveTime, long lastTimeMoved,
        boolean neverRelinquishIdentity, TaskDescription _lastTaskDescription,
        TaskThumbnailInfo lastThumbnailInfo, int taskAffiliation, int prevTaskId,
        int nextTaskId, int taskAffiliationColor, int callingUid, String callingPackage,
        int resizeMode, boolean supportsPictureInPicture, boolean privileged,
        boolean _realActivitySuspended, boolean userSetupComplete, int minWidth,
        int minHeight) {          
}
~~~
<p>TaskRecord.java를 확인해본 결과, TaskRecord는 인자에 따라 3가지 형태를 취하고 있었다.<br>
default 접근자로 설정된 TaskRecord가 다른 클래스에서 사용될 수 있는데, 이 둘을 나눈것은 VoiceInteraction의 사용여부인 것 같다.<br>
VoiceInteraction는 안드로이드 6.0 (마시멜로우) 버전에서 추가된 기능으로 음성 액션을 가능캐하는 기능이다.<br>
6.0 버전 이전에는 그렇다면 2가지 형태를 취하고 있지 않을까?<br>
private 접근자로 설정된 TaskRecord는 가장 상세한 Task정보를 담고있고 이것은 정보 확인을 위해 사용되지 않을까 생각된다.<br>
TaskRecord.java내에는  restoreFromXml()라는 메소드가 존재하는데, restoreFromXml()는 Xml 파일을 읽어 그 정보를 통해 TaskRecord를 만들어 내는 메소드 같다.
</p>

~~~
while (((event = in.next()) != XmlPullParser.END_DOCUMENT) &&
        (event != XmlPullParser.END_TAG || in.getDepth() >= outerDepth)) {
    if (event == XmlPullParser.START_TAG) {
        final String name = in.getName();
        if (TaskPersister.DEBUG) Slog.d(TaskPersister.TAG, "TaskRecord: START_TAG name=" +
                name);
        if (TAG_AFFINITYINTENT.equals(name)) {
            affinityIntent = Intent.restoreFromXml(in);
        } else if (TAG_INTENT.equals(name)) {
            intent = Intent.restoreFromXml(in);
        } else if (TAG_ACTIVITY.equals(name)) {
            ActivityRecord activity = ActivityRecord.restoreFromXml(in, stackSupervisor);
            if (TaskPersister.DEBUG) Slog.d(TaskPersister.TAG, "TaskRecord: activity=" +
                    activity);
            if (activity != null) {
                activities.add(activity);
            }
        } else {
            Slog.e(TAG, "restoreTask: Unexpected name=" + name);
            XmlUtils.skipCurrentTag(in);
        }
    }
}
~~~
<p>위의 코드는 TaskRecord.java내 restoreFromXml() 메소드의 일부이다.<br>
Intent 클래스와 ActivityRecord 클래스에서도 restoreFromXml()이란 메소드가 존재함을 확인할 수 있었다.<br>
restoreFromXml()을 보아 xml에는 작업에 대한 상세한 정보가 담긴 매우 중요한 파일일 것이다.<br>
인텐트와 액티비티, 태스크에도 영향을 줄 수 있는..</p>
