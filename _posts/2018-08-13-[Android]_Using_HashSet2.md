---
layout: post
title: "[180807] IntentFilter의 성능개선(2)"
date:   2018-08-13 17:55:25 +0900
---

IntentFilter.java 내의 matchCategories 메소드

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

***

mCategories가 ArrayList일 경우와 HashSet일 경우 성능 차이를 확인
