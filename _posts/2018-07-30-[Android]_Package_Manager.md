---
layout: post
title: "[180730] PackageManagerService (1)"
date:   2018-07-30 17:55:25 +0900
---

<h3>인텐트 필터 정보를 가진 곳은 어디?</h3>
패키지매니저에서 설치된 앱의 정보를 다룬다고 하는데, 그렇다면 매니페스트의 intent-filter를 파싱해 어디서 저장하고 있는지 알고 싶었다.<br>
그래서 가장 먼저 PackageManagerService.java에서 intentfilter로 검색을 한 결과<br>
IntentResolver를 확장해 ActivityIntentResolver라는 클래스를 새로 정의하고 있었고,<br>
아마도 여기서 앱의 인텐트 필터의 정보를 다룰것으로 생각되어 ActivityIntentResolver의 코드를 중점적으로 보았다.

~~~
public List<ResolveInfo> queryIntentForPackage(Intent intent, String resolvedType,
        int flags, ArrayList<PackageParser.Activity> packageActivities, int userId) {
    if (!sUserManager.exists(userId)) return null;
    if (packageActivities == null) {
        return null;
    }
    mFlags = flags;
    final boolean defaultOnly = (flags & PackageManager.MATCH_DEFAULT_ONLY) != 0;
    final int N = packageActivities.size();
    ArrayList<PackageParser.ActivityIntentInfo[]> listCut =
        new ArrayList<PackageParser.ActivityIntentInfo[]>(N);

    ArrayList<PackageParser.ActivityIntentInfo> intentFilters;
    for (int i = 0; i < N; ++i) {
        intentFilters = packageActivities.get(i).intents;
        if (intentFilters != null && intentFilters.size() > 0) {
            PackageParser.ActivityIntentInfo[] array =
                    new PackageParser.ActivityIntentInfo[intentFilters.size()];
            intentFilters.toArray(array);
            listCut.add(array);
        }
    }
    return super.queryIntentFromList(intent, resolvedType, defaultOnly, listCut, userId);
}
~~~

위의 코드는 ActivityIntentResolver의 내부에 존재하는 메소드인데,<br>
변수명부터 intentFilters라고 되어있고, 마침 ArrayList의 구조를 갖고 있는 것으로 보아 여기서 실마리를 잡을 수 있을 것이라 생각되었다.<br>
intentFilters의 내용을 알기위해선 PackageParser.ActivityIntentInfo가 무엇인지 확인해야 했다.

~~~
public final static class ActivityIntentInfo extends IntentInfo {
    public Activity activity;

    public ActivityIntentInfo(Activity _activity) {
        activity = _activity;
    }

    public String toString() {
        StringBuilder sb = new StringBuilder(128);
        sb.append("ActivityIntentInfo{");
        sb.append(Integer.toHexString(System.identityHashCode(this)));
        sb.append(' ');
        activity.appendComponentShortName(sb);
        sb.append('}');
        return sb.toString();
    }

    public ActivityIntentInfo(Parcel in) {
        super(in);
    }
}
~~~

PackageParser.java 내부의 ActivityIntentInfo 클래스의 모습이다.<br>
IntentInfo를 확장하여 다시 IntentInfo.java를 찾아보려 했지만 같은 경로상에 존재하지 않고,<br>
PackageParser.java 내부에 추상클래스로 정의되어 있는 것을 확인할 수 있었다.

~~~
public static abstract class IntentInfo extends IntentFilter {
    public boolean hasDefault;
    public int labelRes;
    public CharSequence nonLocalizedLabel;
    public int icon;
    public int logo;
    public int banner;
    public int preferred;

    protected IntentInfo() {
    }
    protected IntentInfo(Parcel dest) {
        super(dest);
        hasDefault = (dest.readInt() == 1);
        labelRes = dest.readInt();
        nonLocalizedLabel = dest.readCharSequence();
        icon = dest.readInt();
        logo = dest.readInt();
        banner = dest.readInt();
        preferred = dest.readInt();
    }

    public void writeIntentInfoToParcel(Parcel dest, int flags) {
        super.writeToParcel(dest, flags);
        dest.writeInt(hasDefault ? 1 : 0);
        dest.writeInt(labelRes);
        dest.writeCharSequence(nonLocalizedLabel);
        dest.writeInt(icon);
        dest.writeInt(logo);
        dest.writeInt(banner);
        dest.writeInt(preferred);
    }
}
~~~

IntentFilter 클래스는 같은 경로상에 존재하지 않았고, 이전에 인텐트 필터를 다룬 <a href="https://dhfkfl1.github.io/dhfkfl1.github.io/2018/07/10/Android-_Intent_Filter.html">포스팅</a>에서의 그 IntentFilter.java가 아닐까 싶다.<br>
가장 보고싶었던 것은 매니페스트 파일을 파싱해 intent-filter 부분을 저장하는 모습이 었는데,<br>
지금까지 따라온 것을 되돌아보니 번지수를 잘못 찾은듯하다.<br>

~~~
while ((type=parser.next()) != XmlPullParser.END_DOCUMENT
       && (type != XmlPullParser.END_TAG
               || parser.getDepth() > outerDepth)) {
    if (type == XmlPullParser.END_TAG || type == XmlPullParser.TEXT) {
        continue;
    }

    if (parser.getName().equals("intent-filter")) {
        ActivityIntentInfo intent = new ActivityIntentInfo(a);
        if (!parseIntent(res, parser, true /*allowGlobs*/, true /*allowAutoVerify*/,
                intent, outError)) {
            return null;
        }
        if (intent.countActions() == 0) {
            Slog.w(TAG, "No actions in intent filter at "
                    + mArchiveSourcePath + " "
                    + parser.getPositionDescription());
        } else {
            a.intents.add(intent);
        }
        // adjust activity flags when we implicitly expose it via a browsable filter
        final int visibility = visibleToEphemeral
                ? IntentFilter.VISIBILITY_EXPLICIT
                : isImplicitlyExposedIntent(intent)
                        ? IntentFilter.VISIBILITY_IMPLICIT
                        : IntentFilter.VISIBILITY_NONE;
        intent.setVisibilityToInstantApp(visibility);
        if (intent.isVisibleToInstantApp()) {
            a.info.flags |= ActivityInfo.FLAG_VISIBLE_TO_INSTANT_APP;
        }
        if (intent.isImplicitlyVisibleToInstantApp()) {
            a.info.flags |= ActivityInfo.FLAG_IMPLICITLY_VISIBLE_TO_INSTANT_APP;
        }
    } else if (parser.getName().equals("meta-data")) {
        if ((a.metaData=parseMetaData(res, parser, a.metaData,
                outError)) == null) {
            return null;
        }
    } else {
        if (!RIGID_PARSER) {
            Slog.w(TAG, "Unknown element under <activity-alias>: " + parser.getName()
                    + " at " + mArchiveSourcePath + " "
                    + parser.getPositionDescription());
            XmlUtils.skipCurrentTag(parser);
            continue;
        } else {
            outError[0] = "Bad element under <activity-alias>: " + parser.getName();
            return null;
        }
    }
}
~~~
위의 코드는 PackageParser.java 내의 parseActivityAlias()라는 메소드이다.<br>
자세히 살펴보진 않았지만, 내부의 동작이 마치 내가 찾고있는 것과 같아<br>
다음 포스팅에선 위의 코드를 따라가 볼 생각이다.
