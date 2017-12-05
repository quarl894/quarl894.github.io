---
layout: post
# title:  "Android release APK 생성하기"
date:   2017-12-05 22:20:00 +0900
categories: Android APK
---

App을 완성해서 Goolge Play에 올리려면 APK를 만들어야 한다.
기본으로 Debug.apk는 만들어져 있지만 Debug용은 올릴 수 없다.
따라서, Release용 APK를 만들어야 한다. 이제 만들어 보자.
# 1. Anroid Key Store 생성

Android Project를 생성 후 배포를 하려면 반드시 개발자의 KeyStore 파일을 이용한 서명(Signing)을 해야 한다.
서명은 앱의 개발자임을 증명하는 아주 중요한 수단이다. 때문에 한 번 서명한 KeyStore파일은 계속 백업 후 유지해야 한다.
같은 패키지명을 사용하다 하더라도 서명이 다르면 그 앱은 업데이트가 되지 않는다.
처음에 모르고 다른 Key로 생성된 APK로 업데이트 하려다가 시간만 버린 적이 있다..

## Command로 생성
keytool은 jdk 설치 폴더의 bin에 위치해 있기 때문에 Java를 환경변수설정으로 지정해주었다면 위치와 상관없이 command창에서 실행이 가능하지만 환경변수설정을 지정 하지 않았다면 keytool의 경로로 가서 아래의 커맨드를 입력하도록 한다.

![cmd](https://quarl894.github.io/assets/posts/20171205/cmd.png)

## Androud Studio에서 keystore 생성

- build -Generate Singed APK...
![android](https://quarl894.github.io/assets/posts/20171205/and.jpg)

- Create New... (정보 입력)
![android2](https://quarl894.github.io/assets/posts/20171205/and2.png)

- 다음과정에서 혹시 database password를 입력하라고 나올 때가 있다.
이 경우에는 settings-> system settings -> passwords 이동 후 변경해주면 된다.
![android](https://quarl894.github.io/assets/posts/20171205/and3.png)

- Error:(65) Error: "..." is not translated in "ko" (korean) [MissingTranslation]

 build.gradle에 추가 시키면 예외처리 된다.
```java
android {
  lintOptions {
      checkReleaseBuilds false
  }
}
```

- app 하위 폴더에 app-release.apk가 만들어진다.

- 끝
[jekyll-gh]:   https://github.com/quarl894
