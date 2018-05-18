---
layout: post
title: "Activity Record study"
date:   2018-05-04 15:55:25 +0900
---
<h3>ActivityRecord.java내에서 ActivityRecord의 형태는?</h3>
<p>지난 포스트에선 TaskRecord에 대한 부분을 간략하게 살펴보았다. <br>
오늘은 ActivityRecord의 내용을 살펴볼 것이다.</p>

~~~
ActivityRecord(ActivityManagerService _service, ProcessRecord _caller, int _launchedFromPid,
            int _launchedFromUid, String _launchedFromPackage, Intent _intent, String _resolvedType,
            ActivityInfo aInfo, Configuration _configuration,
            ActivityRecord _resultTo, String _resultWho, int _reqCode,
            boolean _componentSpecified, boolean _rootVoiceInteraction,
            ActivityStackSupervisor supervisor, ActivityOptions options,
            ActivityRecord sourceRecord) {
        service = _service;
        appToken = new Token(this);
        info = aInfo;
        launchedFromPid = _launchedFromPid;
        launchedFromUid = _launchedFromUid;
        launchedFromPackage = _launchedFromPackage;
        userId = UserHandle.getUserId(aInfo.applicationInfo.uid);
        intent = _intent;
        shortComponentName = _intent.getComponent().flattenToShortString();
        resolvedType = _resolvedType;
        componentSpecified = _componentSpecified;
        rootVoiceInteraction = _rootVoiceInteraction;
        mLastReportedConfiguration = new MergedConfiguration(_configuration);
        resultTo = _resultTo;
        resultWho = _resultWho;
        requestCode = _reqCode;
        state = INITIALIZING;
        frontOfTask = false;
        launchFailed = false;
        stopped = false;
        delayedResume = false;
        finishing = false;
        deferRelaunchUntilPaused = false;
        keysPaused = false;
        inHistory = false;
        visible = false;
        nowVisible = false;
        idle = false;
        hasBeenLaunched = false;
        mStackSupervisor = supervisor;

        mRotationAnimationHint = aInfo.rotationAnimation;

        if (options != null) {
            pendingOptions = options;
            mLaunchTaskBehind = pendingOptions.getLaunchTaskBehind();

            final int rotationAnimation = pendingOptions.getRotationAnimationHint();
            // Only override manifest supplied option if set.
            if (rotationAnimation >= 0) {
                mRotationAnimationHint = rotationAnimation;
            }
            PendingIntent usageReport = pendingOptions.getUsageTimeReport();
            if (usageReport != null) {
                appTimeTracker = new AppTimeTracker(usageReport);
            }
        }

        // This starts out true, since the initial state of an activity is that we have everything,
        // and we shouldn't never consider it lacking in state to be removed if it dies.
        haveState = true;

        // If the class name in the intent doesn't match that of the target, this is
        // probably an alias. We have to create a new ComponentName object to keep track
        // of the real activity name, so that FLAG_ACTIVITY_CLEAR_TOP is handled properly.
        if (aInfo.targetActivity == null
                || (aInfo.targetActivity.equals(_intent.getComponent().getClassName())
                && (aInfo.launchMode == LAUNCH_MULTIPLE
                || aInfo.launchMode == LAUNCH_SINGLE_TOP))) {
            realActivity = _intent.getComponent();
        } else {
            realActivity = new ComponentName(aInfo.packageName, aInfo.targetActivity);
        }
        taskAffinity = aInfo.taskAffinity;
        stateNotNeeded = (aInfo.flags & FLAG_STATE_NOT_NEEDED) != 0;
        appInfo = aInfo.applicationInfo;
        nonLocalizedLabel = aInfo.nonLocalizedLabel;
        labelRes = aInfo.labelRes;
        if (nonLocalizedLabel == null && labelRes == 0) {
            ApplicationInfo app = aInfo.applicationInfo;
            nonLocalizedLabel = app.nonLocalizedLabel;
            labelRes = app.labelRes;
        }
        icon = aInfo.getIconResource();
        logo = aInfo.getLogoResource();
        theme = aInfo.getThemeResource();
        realTheme = theme;
        if (realTheme == 0) {
            realTheme = aInfo.applicationInfo.targetSdkVersion < HONEYCOMB
                    ? android.R.style.Theme : android.R.style.Theme_Holo;
        }
        if ((aInfo.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
            windowFlags |= LayoutParams.FLAG_HARDWARE_ACCELERATED;
        }
        if ((aInfo.flags & FLAG_MULTIPROCESS) != 0 && _caller != null
                && (aInfo.applicationInfo.uid == SYSTEM_UID
                    || aInfo.applicationInfo.uid == _caller.info.uid)) {
            processName = _caller.processName;
        } else {
            processName = aInfo.processName;
        }

        if ((aInfo.flags & FLAG_EXCLUDE_FROM_RECENTS) != 0) {
            intent.addFlags(FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS);
        }

        packageName = aInfo.applicationInfo.packageName;
        launchMode = aInfo.launchMode;

        Entry ent = AttributeCache.instance().get(packageName,
                realTheme, com.android.internal.R.styleable.Window, userId);
        fullscreen = ent != null && !ActivityInfo.isTranslucentOrFloating(ent.array);
        noDisplay = ent != null && ent.array.getBoolean(
                com.android.internal.R.styleable.Window_windowNoDisplay, false);

        setActivityType(_componentSpecified, _launchedFromUid, _intent, options, sourceRecord);

        immersive = (aInfo.flags & FLAG_IMMERSIVE) != 0;

        requestedVrComponent = (aInfo.requestedVrComponent == null) ?
                null : ComponentName.unflattenFromString(aInfo.requestedVrComponent);

        mShowWhenLocked = (aInfo.flags & FLAG_SHOW_WHEN_LOCKED) != 0;
        mTurnScreenOn = (aInfo.flags & FLAG_TURN_SCREEN_ON) != 0;
    }
~~~
<p>어제 살펴본 TaskRecord와 달리 ActivityRecord는 한 종류의 생성자만 존재했다.<br>
VoiceInteraction에 대한 인자를 받는 것으로 보아 TaskRecord와 달리 ActivityRecord는 6.0 버전 이후 최신화가 이루어진 파일이 아닐까 생각이 든다.<br>
변수의 내용에서도 icon, logo, theme같은 부분은 어플리케이션의 정보를 가져와 저장하는 듯하다.<br>
fullscreen, noDisplay같은 변수를 보아 화면 구성에 대한 정보도 ActivityRecord에 존재하는 것 같다.
Flag의 이름을 보아도 화면에 대한 제어가 이루어짐을 짐작할 수 있다.
</p>
