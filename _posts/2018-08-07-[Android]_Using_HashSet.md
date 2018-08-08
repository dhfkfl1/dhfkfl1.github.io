---
layout: post
title: "[180807] IntentFilter의 성능개선 가능성"
date:   2018-08-07 17:55:25 +0900
---

IntentFilter.java 내의 matchCategories 메소드이다.

~~~
public final String matchCategories(Set<String> categories) {
    if (categories == null) {
        return null;
    }
    Iterator<String> it = categories.iterator();

    if (mCategories == null) {
        return it.hasNext() ? it.next() : null;
    }
    while (it.hasNext()) {
        final String category = it.next();
        if (!mCategories.contains(category)) {
            return category;
        }
    }
    return null;
}
~~~

While 내부 코드를 바꿔 matchCategories 메소드의 성능을 향상시키면,
안드로이드의 성능도 개선될 것이다.

1. matchCategories 메소드가 얼마나 자주 사용되는지
IntentFilter.java 내의 match 메소드 내에서만 사용된다.
~~~
public final int match(String action, String type, String scheme,
        Uri data, Set<String> categories, String logTag) {
    if (action != null && !matchAction(action)) {
        if (false) Log.v(
            logTag, "No matching action " + action + " for " + this);
        return NO_MATCH_ACTION;
    }

    int dataMatch = matchData(type, scheme, data);
    if (dataMatch < 0) {
        if (false) {
            if (dataMatch == NO_MATCH_TYPE) {
                Log.v(logTag, "No matching type " + type
                      + " for " + this);
            }
            if (dataMatch == NO_MATCH_DATA) {
                Log.v(logTag, "No matching scheme/path " + data
                      + " for " + this);
            }
        }
        return dataMatch;
    }

    String categoryMismatch = matchCategories(categories);
    if (categoryMismatch != null) {
        if (false) {
            Log.v(logTag, "No matching category " + categoryMismatch + " for " + this);
        }
        return NO_MATCH_CATEGORY;
    }

    // It would be nice to treat container activities as more
    // important than ones that can be embedded, but this is not the way...
    if (false) {
        if (categories != null) {
            dataMatch -= mCategories.size() - categories.size();
        }
    }

    return dataMatch;
}
~~~

2. IntentFilter.java 내의 match 메소드가 사용되는 곳
matchDataAuthority(), hasDataPath(), hasDataAuthority(), hasDataSchemeSpecificPart(), equals() 등의 메소드 내에서 사용된다.

***
