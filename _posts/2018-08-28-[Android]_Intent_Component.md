---
layout: post
title: "[180828] Intent의 속성들"
date:   2018-08-28 17:55:25 +0900
---

~~~
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
~~~

Intent.java 내에 정의된 인텐트 객체의 속성들이다.<br>
이전 포스팅에선 다루지않았던 속성들에 대해서 알아보았다.

---

1. mSourceBounds
Rect이란 타입으로 정의되어 있는 이것은 화면의 사각형의 크기와 위치를 표현하기 위한 객체라고 한다.<br>

~~~
public final class Rect implements Parcelable {
    public int left;
    public int top;
    public int right;
    public int bottom;

    /**
     * Create a new empty Rect. All coordinates are initialized to 0.
     */
    public Rect() {}

    /**
     * Create a new rectangle with the specified coordinates. Note: no range
     * checking is performed, so the caller must ensure that left <= right and
     * top <= bottom.
     *
     * @param left   The X coordinate of the left side of the rectangle
     * @param top    The Y coordinate of the top of the rectangle
     * @param right  The X coordinate of the right side of the rectangle
     * @param bottom The Y coordinate of the bottom of the rectangle
     */
    public Rect(int left, int top, int right, int bottom) {
        this.left = left;
        this.top = top;
        this.right = right;
        this.bottom = bottom;
    }
~~~
/framework/base/graphics/java/android/graphics/Rect.java<br>
사각형을 표현하기 위한 클래스라는 것은 알겠으나 인텐트 객체에 이러한 정보를 추가해 어떤 기능을 수행할지는 잘 모르겠다.<br>
mSourceBounds를 사용하지 않고, mExtras에 화면의 위치정보를 담을 수 있다고 알고있는데 그 차이는 Rect이 제공하는 편리한 메소드가 있어서 그런것 일까?

2. mSelector
암시적 인텐트는 특정 구성 요소의 이름을 대지 않고, 수행할 일반적인 작업을 선언하여 다른 앱의 구성 요소가 이를 처리하게 한다.<br>
여러 앱이 처리가능한 경우, 아래와 같이 어떤 앱으로 수행할 것인지 선택할 수 있다.<br>
<img src="/assets/images/ss.png">
