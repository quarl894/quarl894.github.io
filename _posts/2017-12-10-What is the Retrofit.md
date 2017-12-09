---
layout: post
# title:  "What is the Retrofit"
date:   2017-12-10 01:20:00 +0900
categories: Retrofit, Rest API
---
# Retrofit 정의
- HTTP API를 자바 인터페이스 형태로 사용할 수 있도록 만들어 놓은 라이브러리.

# 기존 통신의 문제점 (AsyncTask,HttpUrlConnection)

1. 네트워크 통신 연결/해제
2. 가져온 데이터 파싱
3. Json통신의 경우, Json데이터<-> Class 변환
4. 각종 에러처리

# Retrofit Function

retrofit은 Rest API에서 사용하는 CURD 기능을 모두 제공합니다.
POST (create) , PUT (update), GET (read), DELETE (delete) 또한 annotation (@)을 활용하여 손쉽게 통신 코드를 작성할 수 있습니다.
현재 가장 많이 사용되는 JSON이나 XML 방식의 데이터를 지원하는 라이브러리를 가지고 있습니다.

# Retrofit 장점

## 1. 낮은 비용

기존의 Asynctask를 활용한 통신을 사용해 보셨나요? 구현하는데 굉장히 불편한 것을 느끼셨을 것입니다. 저도 몇 차례 사용해 봤지만, 외우지 못하고 항상 블로그를 참고하며 사용했던 기억이 있습니다. 그만큼 불편하고 시간을 소요한다는 것이죠.
반면 retrofit은 익히는데 시간은 썼지만, 굉장히 간단하게 사용할 수 있다는 것을 알 수 있었습니다.
　
## 2. 재사용성

마찬가지로, 기존의 코드는 재사용하기 매우 불편했습니다. 물론 그 구조를 재사용 가능하게 모듈화 하시는 분도 있겠지만, 저를 비롯한 일반적인 개발자들에겐 불편하고 어려운 일이라고 생각합니다. retrofit은 주고 받을 데이터 형식 등의 수정으로 쉽게 재사용이 가능합니다.
　
## 3. 유지보수

재사용성과 마찬가지라고 보셔도 될 것 같습니다. 제가 retrofit을 안드로이드에서 구현한 것을 참고하시면 좋겠습니다만, 이미 통신 구조나, 자료형 같은 부분의 모듈화가 되어있습니다. 기능 추가 및 삭제는 너무 간단하죠.
　
## 4. 가독성

pair programming 이나, git을 통한 프로젝트 공유가 많은 현재 개발 상황에서, 가독성은 매우 중요한 문제입니다. retrofit은 주석 없이도 이해할 수 있으실 겁니다.
　
## 5. 성능
개발자에게 가장 중요한 부분은 바로 이 부분일 것입니다. 좋은 성능을 보인다면, 다소 복잡하고 어렵더라도 쓰게 되고, 쓰고 싶어지죠! 아래 표를 보시겠습니다.

![vs](http://quarl894.github.io/assets/posts/20171210/vs.png)

압도적인 성능 차이를 보여주는 것을 확인할 수 있습니다. 기존보다 3~10배 이상의 성능이라면, 자바 & 안드로이드 개발자에겐 이미 필수적인 요소라고 생각합니다.

[예제코드](https://github.com/quarl894/Retrofit_example.git)

[jekyll-gh]:   https://github.com/quarl894