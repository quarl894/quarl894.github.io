---
layout: post
# title:  "Android Oreo에서 변경된 기능"
date:   2018-02-13 22:20:00 +0900
categories: Android oreo 기술면접
tag: Android8.0
---

먼저 변경사항이 너무 많아서 제 기준 우선적으로 사용하고 필요하다고 느끼는 것들만 정리했습니다.

Android 8.0에서는 배터리 수명과 개인정보를 중점적으로 강화했습니다.

## 알림

#### 알림 채널

> Android 8.0에서는 알림 채널이 도입됩니다. 알림 채널은 개발자가 표시하고자 하는 각 유형의 알림에 대해 사용자 맞춤형 채널을 생성합니다. 사용자 인터페이스에서는 알림 채널을 알림 범주라고 부릅니다.

다시 알림, 알림 제한 시간, 알림 설정, 배경 색상, 메세징 스타일 등의 기능을 사용자에 맞게 개발자가 변경할 수 있습니다.

**오레오에서는 알림 채널을 구성하지 않으면 알림이 가지 않습니다.!**

## Autofill Framework

> 로그인 및 신용카드 양식 등과 같은 양식을 더 쉽게 작성할 수 있도록 지원합니다. 사용자가 Autofill에 옵트인하면 기존 앱과 새로운 앱에서 Autofill Framework를 사용할 수 있습니다.

## Fonts

#### 다운 가능한 글꼴

> Android 8.0와 Android 지원 라이브러리 26을 사용하면, 글꼴을 APK에 번들링하거나 APK로 글꼴을 다운로드하는 대신 글꼴을 제공자 애플리케이션에게 요청할 수 있습니다. 이 기능은 APK 크기를 줄여주고, 앱 설치 성공률을 높여주며, 동일 글꼴을 여러 앱이 공유하도록 허용합니다.

#### XML 글꼴

> Android 8.0는 글꼴을 리소스로 사용할 수 있는 Fonts in XML이라는 새로운 기능을 도입합니다. 즉, 글꼴을 자산으로 번들할 필요가 없습니다. 글꼴은 R 파일로 컴파일되고 시스템에서 자동으로 리소스로 사용할 수 있습니다. 그런 다음 새로운 리소스 유형인 font의 도움을 통해 해당 글꼴에 액세스할 수 있습니다.

지원 라이브러리 26은 API 버전 14 이상이 실행되는 기기에서 이 기능을 완벽하게 지원합니다.

## TextView 자동 크기 조절

> Android 8.0에서는 TextView 크기에 따라 텍스트의 크기를 자동으로 늘리거나 줄일 수 있습니다. 다시 말해, 다양한 화면에서 또는 동적 콘텐츠로 텍스트 크기를 더 쉽게 최적화할 수 있습니다.

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform" />
```
android:autoSizeTextType으로 동적으로 크기를 최적하시켜줍니다.

**사이즈 default값: minTextSize = 12sp, maxTextSize = 112sp, and granularity = 1px.**

## WebView API

#### 제공 API

- Version API
- Google SafeBrowsing API
- Termination Handle API
- Renderer Importance API

주로 Video, size, zoom 등 기술적으로 지원이 늘어났습니다.

또한 보안강화에도 신경을 썼습니다.

#### 보안

> 앱의 네트워크 보안 구성에서 지원되는 일반 텍스트 트래픽을 옵트아웃하는 경우, 이 앱의 WebView 객체가 HTTP를 통해 웹사이트에 액세스할 수 없습니다. 각 WebView 객체는 대신 HTTPS를 사용해야 합니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">secure.example.com</domain>
    </domain-config>
</network-security-config>
```
NetworkSecurityPolicy.isCleartextTrafficPermitted()를 통해서 신뢰가 가지 않는 사이트는 Https로만 수행가능하게 합니다.


## findViewById() 서명 변경

> findViewById() 메서드의 모든 인스턴스는 View 대신 <T extends View> T를 반환합니다. 이 변경은 다음과 같은 의미를 지닙니다.

기존 코드에 모호한 반환 유형이 있을 수 있습니다(예: someMethod(View) 및 someMethod(TextView)가 둘 다 findViewById() 호출 결과를 취하는 경우).

Java 8 소스 언어를 사용하는 경우, 반환 유형이 제한되지 않는다면 View에 대한 명시적 형변환이 필요합니다.(예:assertNotNull(findViewById(...)).someViewMethod())).

final이 아닌 findViewById() 메서드(예: Activity.findViewById())를 재정의하는 경우 그 반환 유형을 업데이트해야 합니다.

```java
btn = (Button) findViewById(R.id.btn);
btn = findViewById(R.id.btn);
```

기존에 첫번째 코드처럼 썼지만 이제는 두번째 코드처럼 명시적으로 선언해줄 필요가 없습니다.

## 스마트 공유

> Android 8.0는 사용자의 맞춤설정된 공유 기본 설정에 대한 정보를 획득하여 공유할 앱의 각 콘텐츠 유형에 대해 더 정확히 파악합니다. 예를 들어, 사용자가 영수증 사진을 찍으면 Android 8.0가 지출 추적 앱을 추천할 수 있고, 사용자가 셀카를 찍으면 소셜 미디어 앱이 이미지를 더욱 효과적으로 처리할 수 있습니다. Android 8.0는 사용자의 맞춤설정된 기본 설정에 따라 이런 패턴을 전부 자동으로 학습합니다.

스마트 공유는 audio, video, text, URL 등과 같이, image 이외의 콘텐츠 유형에 효과가 있습니다.

스마트 공유를 사용하려면 콘텐츠를 공유하는 인텐트에 최대 3개의 문자열 주석으로 구성된 ArrayList를 추가합니다. 주석은 콘텐츠의 주요 구성 요소나 항목을 설명하는 내용이어야 합니다. 다음 코드 예시는 인텐트에 주석을 추가하는 방법을 보여줍니다.

```java
ArrayList<String> annotations = new ArrayList<>();

annotations.add("topic1");
annotations.add("topic2");
annotations.add("topic3");

intent.putStringArrayListExtra(
    Intent.EXTRA_CONTENT_ANNOTATIONS,
    annotations
);
```

[참고사이트](https://developer.android.com/about/versions/oreo/android-8.0.html?hl=ko)

[jekyll-gh]:   https://github.com/quarl894