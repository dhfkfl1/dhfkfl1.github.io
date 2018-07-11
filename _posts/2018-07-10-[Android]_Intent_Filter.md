---
layout: post
title: "[180710] 인텐트의 유형과 인텐트 필터"
date:   2018-07-10 17:55:25 +0900
---

<h3>인텐트 유형</h3>
<br>
- 명시적 인텐트<br>
인텐트에 클래스 객체나 컴포넌트 이름을 지정하여 호출할 대상을 명확히 알 수 있는 경우에 사용한다.<br>
주로 본인의 앱 내부에서 사용하는 인텐트 유형으로, 특정 컴포넌트나 액티비티가 명확하게 실행되어야할 경우 사용한다.<br>
setComponent()로 인텐트를 생성하는 경우가 명시적 인텐트에 해당한다.<br>


- 암시적 인텐트<br>
인텐트에 특정 구성 요소의 이름을 지정하지 않고, 액션과 데이터를 지정하여 다른 구성 요소가 이를 처리할 수 있게 한다.<br>
setAction(), setCategory()와 같은 메서드로 인텐트를 생성하는 경우가 암시적 인텐트에 해당한다.<br>
별도로 Action, Category, Type을 설정한 인텐트에 Component가 설정되면 (setComponent()) 기존에 셋팅한 정보를 무시하게 된다.<br>

<img src="/assets/images/Intent.JPG" width="1000" height="350">
<a href="https://developer.android.com/guide/components/intents-filters#Types">이미지 출처</a>

<br>
  1. 액티비티 A가 작업을 명시한 인텐트를 생성하여 이를 startActivity()에 전달<br>
  2. 안드로이드 시스템이 해당 인텐트와 일치하는 인텐트 필터를 찾아 모든 앱을 검색<br>
  3. 시스템이 일치하는 액티비티(B)를 시작하기 위해 그 액티비티의 onCreate() 메서드를 호출하여 이를 인텐트에 전달<br>



<h3>인텐트 필터</h3><br>
인텐트 필터는 앱 컴포넌트가 받고자하는 인텐트가 무엇인지를 정하는 수단이다.<br>
명시적 인텐트는 실행할 컴포넌트를 명확히 하여 실행될 컴포넌트가 명확하지만, 암시적 인텐트는 컴포넌트 이름이 아닌 액션과 데이터를 설정하여 어떤 컴포넌트가 실행될 지 명확하지 않다.<br>
때문에, 안드로이드의 앱 컴포넌트는 인텐트 필터를 가질 수 있고, 이를 이용하여 적절한 인텐트를 전달하게 된다.<br>


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
    if (false) {
        if (categories != null) {
            dataMatch -= mCategories.size() - categories.size();
        }
    }
    return dataMatch;
}
~~~

IntentFilter.java의 많은 메서드 가운데 가장 핵심이 되는 메서드는 match()가 아닐까 싶다.<br>
match()를 어디서 호출하는지는 찾지 못했지만 단순히 코드를 보고 내가 이해한 과정은 이렇다.<br>
암시적 인텐트를 받은 안드로이드 시스템은 모든 앱의 manifest.xml에서 인텐트 필터 영역을 match() 메서드를 이용해 인텐트 내부의 설정값과 manifest.xml의 인텐트 필터 영역을 매칭한다.<br>
매칭은 Action, Data, Category 필드 순으로 진행되며 불일치 시 로그를 남긴다.<br>
모든 필드가 매칭되는 컴포넌트에 한하여 인텐트를 전달한다.<br>




~~~
public final int matchData(String type, String scheme, Uri data) {
       final ArrayList<String> types = mDataTypes;
       final ArrayList<String> schemes = mDataSchemes;

       int match = MATCH_CATEGORY_EMPTY;

       if (types == null && schemes == null) {
           return ((type == null && data == null)
               ? (MATCH_CATEGORY_EMPTY+MATCH_ADJUSTMENT_NORMAL) : NO_MATCH_DATA);
       }
       if (schemes != null) {
           if (schemes.contains(scheme != null ? scheme : "")) {
               match = MATCH_CATEGORY_SCHEME;
           } else {
               return NO_MATCH_DATA;
           }
           final ArrayList<PatternMatcher> schemeSpecificParts = mDataSchemeSpecificParts;
           if (schemeSpecificParts != null && data != null) {
               match = hasDataSchemeSpecificPart(data.getSchemeSpecificPart())
                       ? MATCH_CATEGORY_SCHEME_SPECIFIC_PART : NO_MATCH_DATA;
           }
           if (match != MATCH_CATEGORY_SCHEME_SPECIFIC_PART) {
               final ArrayList<AuthorityEntry> authorities = mDataAuthorities;
               if (authorities != null) {
                   int authMatch = matchDataAuthority(data);
                   if (authMatch >= 0) {
                       final ArrayList<PatternMatcher> paths = mDataPaths;
                       if (paths == null) {
                           match = authMatch;
                       } else if (hasDataPath(data.getPath())) {
                           match = MATCH_CATEGORY_PATH;
                       } else {
                           return NO_MATCH_DATA;
                       }
                   } else {
                       return NO_MATCH_DATA;
                   }
               }
           }
           if (match == NO_MATCH_DATA) {
               return NO_MATCH_DATA;
           }
       } else {
           if (scheme != null && !"".equals(scheme)
                   && !"content".equals(scheme)
                   && !"file".equals(scheme)) {
               return NO_MATCH_DATA;
           }
       }

       if (types != null) {
           if (findMimeType(type)) {
               match = MATCH_CATEGORY_TYPE;
           } else {
               return NO_MATCH_TYPE;
           }
       } else {
           if (type != null) {
               return NO_MATCH_TYPE;
           }
       }
       return match + MATCH_ADJUSTMENT_NORMAL;
   }
~~~
match() 내부의 matchData() 메서드


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
match() 내부의 matchCategories() 메서드
