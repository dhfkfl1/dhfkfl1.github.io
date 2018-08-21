---
layout: post
title: "[180820] FilterEquals()"
date:   2018-08-20 17:55:25 +0900
---

지난 주 <a href="https://sjyoo1699.github.io/jekyll/update/2018/08/12/WHERE-TO-RUN-THE-ACTIVITY-06.html">팀원의 발표</a>에서 언급된 intent.FilterEquals()에 대해 알아보았다.

~~~
public void callActivityOnCreate(Activity activity, Bundle icicle) {
    if (mWaitingActivities != null) {
        synchronized (mSync) {
            final int N = mWaitingActivities.size();
            for (int i=0; i<N; i++) {
                final ActivityWaiter aw = mWaitingActivities.get(i);
                final Intent intent = aw.intent;
                if (intent.filterEquals(activity.getIntent())) {
                    aw.activity = activity;
                    mMessageQueue.addIdleHandler(new ActivityGoing(aw));
                }
            }
        }
    }

    activity.performCreate(icicle);

    if (mActivityMonitors != null) {
        synchronized (mSync) {
            final int N = mActivityMonitors.size();
            for (int i=0; i<N; i++) {
                final ActivityMonitor am = mActivityMonitors.get(i);
                am.match(activity, activity, activity.getIntent());
            }
        }
    }
}
~~~

FilterEquals() 메소드는 Intent.java에서 찾아볼 수 있었다.

~~~
public boolean filterEquals(Intent other) {
    if (other == null) {
        return false;
    }
    if (!Objects.equals(this.mAction, other.mAction)) return false;
    if (!Objects.equals(this.mData, other.mData)) return false;
    if (!Objects.equals(this.mType, other.mType)) return false;
    if (!Objects.equals(this.mPackage, other.mPackage)) return false;
    if (!Objects.equals(this.mComponent, other.mComponent)) return false;
    if (!Objects.equals(this.mCategories, other.mCategories)) return false;

    return true;
}
~~~

filterEquals() 메소드는 인텐트 객체의 속성들을 비교하며 모두 일치할 경우 true를 반환한다.<br>
ture일 경우, 두 인텐트가 생성한 액티비티는 동일한 액티비티로 여기는 듯하다.

~~~
// ---------------------------------------------------------------------

private String mAction;
private Uri mData;
private String mType;
private String mPackage;
private ComponentName mComponent;
private int mFlags;
private ArraySet<String> mCategories;
private Bundle mExtras;
private Rect mSourceBounds;
private Intent mSelector;
private ClipData mClipData;
private int mContentUserHint = UserHandle.USER_CURRENT;
/** Token to track instant app launches. Local only; do not copy cross-process. */
private String mLaunchToken;

// ---------------------------------------------------------------------
~~~

intent.java 내의 인텐트 객체의 속성들<br>
이전 인텐트 객체의 구성요소에 대한 <a href="https://dhfkfl1.github.io/dhfkfl1.github.io/2018/07/03/Android-_Intent.html">포스팅</a>

---
액티비티의 생성은 startActivity() 메소드에 인자로 인텐트를 전달해 이루어진다.<br>
때문에 인텐트 객체가 같을 경우, 같은 액티비티를 생성할 것이고, 위의 filterEquals() 메소드는 두 인텐트의 속성값을 비교함으로써 액티비티가 동일한 것인지 아닌지를 판단하게 된다.
