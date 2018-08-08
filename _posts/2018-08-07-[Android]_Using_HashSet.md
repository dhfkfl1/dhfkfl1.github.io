---
layout: post
title: "[180807] IntentFilter의 성능개선 가능성"
date:   2018-08-07 17:55:25 +0900
---

IntentFilter.java 내의 matchCategories 메소드와 mCategories의 선언부.

~~~
private ArrayList<String> mCategories = null;

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

matchCategories()는 카테고리의 Mismatch가 있을 경우, 그 값을 반환한다.<br>
카테고리가 모두 일치하면 null을 반환
인자의 Set<String> categories는 인텐트 내부에 포함된 정보를 의미


~~~
public final void addCategory(String category) {
    if (mCategories == null) mCategories = new ArrayList<String>();
    if (!mCategories.contains(category)) {
        mCategories.add(category.intern());
    }
}
~~~

mCategories는 String 타입의 ArrayList로 저장되고 addCategory() 메소드에 의해
값이 추가

***

~~~
public void readFromXml(XmlPullParser parser) throws XmlPullParserException,
        IOException {
    String autoVerify = parser.getAttributeValue(null, AUTO_VERIFY_STR);
    setAutoVerify(TextUtils.isEmpty(autoVerify) ? false : Boolean.getBoolean(autoVerify));

    int outerDepth = parser.getDepth();
    int type;
    while ((type=parser.next()) != XmlPullParser.END_DOCUMENT
           && (type != XmlPullParser.END_TAG
                   || parser.getDepth() > outerDepth)) {
        if (type == XmlPullParser.END_TAG
                || type == XmlPullParser.TEXT) {
            continue;
        }

        String tagName = parser.getName();
        if (tagName.equals(ACTION_STR)) {
            String name = parser.getAttributeValue(null, NAME_STR);
            if (name != null) {
                addAction(name);
            }
        } else if (tagName.equals(CAT_STR)) {
            String name = parser.getAttributeValue(null, NAME_STR);
            if (name != null) {
                addCategory(name);
            }
        }
        XmlUtils.skipCurrentTag(parser);
    }
}
~~~

addCategory 메소드는 readFromXml 메소드 내부에서 호출되어 사용된다.<br>


***

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


***

2. IntentFilter.java 내의 match 메소드가 사용되는 곳

matchDataAuthority(), hasDataPath(), hasDataAuthority(), hasDataSchemeSpecificPart(), equals() 등의 메소드 내에서 사용된다.

***

3.
