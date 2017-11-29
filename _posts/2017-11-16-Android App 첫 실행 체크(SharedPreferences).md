---
layout: post
# title:  "Android App 첫 실행 체크(SharedPreferences)"
date:   2017-11-16 01:20:58 +0900
categories: android
---
App 개발 시에 맨 처음만 보여주는 코치마크나 이용약관 등 처음만 보여주고 다음부턴 보여주지 않게 하는 경우가 있습니다.

그 때 필요한 것인 SharedPreferences입니다. 
DB에 저장되지 않고 Device 메모리에 저장하는 클래스입니다.

먼저 선언부에 SharedPreferences를 선언합니다.
```java
public SharedPreferences prefs;
```

정의부에 prefs라는 이름으로 정의해줍니다.(변수명은 바꿔도 무관함)
```java
prefs = getSharedPreferences("Pref", MODE_PRIVATE);
```

사용 메소드 정의

```java
public void checkFirstRun(){
    boolean isFirstRun = prefs.getBoolean("isFirstRun",true);
    if(isFirstRun)
    {
        Intent newIntent = new Intent(MapsActivity.this, GuideActivity.class);
        startActivity(newIntent);

        prefs.edit().putBoolean("isFirstRun",false).apply();
        //처음만 true 그다음부터는 false 바꾸는 동작
    }
}
```

매우 간단하게 완성했습니다.
코치마크 할 경우에는 frgment와 CircleIndicator를 사용하여 고급스럽게 만들 수 있습니다.

[jekyll-gh]:   https://github.com/quarl894

