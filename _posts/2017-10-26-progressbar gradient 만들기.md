---
layout: post
# title:  "Android progressbar gradient 만들기"
date:   2017-10-26 20:041:58 +0900
categories: android
---
안드로이드 프로젝트 중 프로그레스바에 그라데이션 색을 적용시키는 작업이 있었다.
단색으로만 해봤던 나에겐 처음 해보는 일이여서 먼저 오픈 라이브러리를 찾았다.
그 중에 그라데이션이 적용되는 것도 있었지만 terminate를 true로만 해야 적용이 되어서 다시 찾게되었다.
찾던 중 stackflow에 아주 좋은 정보를 찾게 되었다.

1. 막대 프로그레스바와 원형 프로그레스바의 그라데이션을 적용시키는 것은 다르다.
2. custom xml을 따로 생성해서 style을 주는것이 훨씬 편하다.



###막대 프로그레스바 그라데이션 주기.
1. drawable에 xml로 만들기.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:id="@android:id/background">
        <shape>
            <solid android:color="@color/light_gray"/>
        </shape>
    </item>

    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <gradient
                    android:endColor="@color/p_color2"
                    android:startColor="@color/p_color1"
                    />
            </shape>
        </clip>
    </item>

</layer-list>
```

2. 해당 layout에 progressbar 적용시키기.
```java
    <ProgressBar
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerInParent="true"
        android:indeterminate="false"
        android:max="100"
        android:progress="100"
        android:progressDrawable="@drawable/circle_progress_background" />
```

3. 해당 클래스에 알맞게 thread를 사용하여 프로그래스바 적용시키면 됩니다.

###원형 프로그레스바 그라데이션 주기.(원형은 프로그레스바가 2개 필요!!)
1. drawable에 xml로 만들기.(첫번째는 background나타낼 프로그레스바)

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item android:id="@android:id/progress">
        <shape
            android:innerRadius="60dp"
            android:shape="ring"
            android:thickness="7dp">
            
            <gradient
                android:endColor="@color/light_gray"
                android:startColor="@color/light_gray"
                android:type="sweep" />   
        </shape>
    </item>
</layer-list>
```

foreground 나타낼 프로그레스바(나타낼 그라데이션 색을 적으면 된다)
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item android:id="@android:id/progress">
            <shape
                android:innerRadius="60dp"
                android:shape="ring"
                android:thickness="7dp"
                android:useLevel="true">
                
                <gradient
                    android:endColor="@color/p_color2"
                    android:startColor="@color/p_color1"
                    android:type="sweep" />   
            </shape>
    </item>
</layer-list>
```

2. layout에 적용시키기
```java
     <ProgressBar
        android:id="@+id/circle_progress_bar"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerInParent="true"
        android:max="100"
        android:rotation="-90"
        android:indeterminate="false"
        android:progressDrawable="@drawable/circle_progress_foreground" />
```

3. 동일하게 클래스에 알맞게 적용시키기.(여기서는 freground 프로그레스만 움직이므로 그것만 적용하면 된다)

[jekyll-gh]:   https://github.com/quarl894
