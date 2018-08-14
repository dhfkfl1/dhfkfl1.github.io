---
layout: post
title: "[180813] IntentFilter의 성능개선(2)"
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

mCategories가 ArrayList일 경우와 HashSet일 경우 성능 차이를 확인하기 위해<br>
아래의 코드를 사용

~~~
private static Object[] Integer;

private static ArrayList<String> mCategory = new ArrayList<String>();  

private static Set<String> mCH = new HashSet<String>();

public static void main(String[] args) {

  long start = 0;
  long end = 0;

  int Size = 10000;                                    // 원소의 개수
  int CallNum = 1;                                     // 함수 호출 횟수
  String str = null;

  Set<String> categories = new HashSet<String>();                     


  for(int i = 0; i < Size; i++) {
    categories.add(String.valueOf(6*i));             // HashSet에 원소 삽입
    mCategory.add(String.valueOf(i));                // ArrayList에 원소 삽입
  }

  for(int j = 0; j < mCategory.size(); j++) {          // ArrayList을 HashSet으로 만듦
    mCH.add(mCategory.get(j));
  }


  start = System.currentTimeMillis();                  // 시작시간

  str = matchCategoryArrayList(categories);
//	str = matchCategoryHashSet(categories);

  end = System.currentTimeMillis();                    // 끝나는 시간

  System.out.println(str);
  System.out.println(((double) (end - start) / 1000) + "(sec)");


}

public static String matchCategoryArrayList(Set<String> categories) {   //ArrayList를 사용한 match
  Iterator<String> it = categories.iterator();

  while(it.hasNext()) {
    String st = it.next();
    if(!mCategory.contains(st)) {
      //System.out.println(st);
      return st;
    }
  }
  return null;
}


public static String matchCategoryHashSet(Set<String> categories) {     //HashSet을 사용한 match
  Iterator<String> it = categories.iterator();

  while(it.hasNext()) {
    String st = it.next();
    if(!mCH.contains(st)) {
      //System.out.println(st);
      return st;
    }
  }
  return null;
}
~~~

***

<h3>원소의 개수 차이에 의한 결과</h3>

조건 : 메소드의 호출 1회, mCategory와 categories의 모든 원소가 일치(최악)
<img src="/assets/images/size_w.JPG">


조건 : 메소드 호출 1회, mCategory와 categories의 원소가 일치하지 않음
<img src="/assets/images/size_a.JPG">

<h3>호출 횟수 차이에 의한 결과</h3>

조건 : 원소의 개수 100개, mCategory와 categories의 모든 원소가 일치(최악)
<img src="/assets/images/call_w.JPG">

조건 : 원소의 개수 100개, mCategory와 categories의 원소가 일치하지 않음
<img src="/assets/images/call_a.JPG">

***

원소의 수가 증가할수록 ArrayList와 HashSet의 성능차이가 나타남
HashSet이 더 우수한 성능을 보여줌
<br>
호출 횟수가 증가할수록 ArrayList와 HashSet의 성능차이가 나타남
HashSet이 더 우수한 성능을 보여줌
