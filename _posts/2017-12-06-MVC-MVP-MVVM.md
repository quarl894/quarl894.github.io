---
layout: post
# title:  "MVC-MVP-MVVM"
date:   2017-12-06 12:20:00 +0900
categories: Android 디자인패턴
---

# MVC vs MVP vs MVVM

- Model : 일종의 데이터라 생각하면 됨. 데이터 이외에 데이터를 조작하는 간단한 로직이 추가되기도 한다.

- View : 사용자에게 보여지는 UI layout

### 이러한 디자인 패턴들이 나오게 된 궁극적인 이유는?
-> 각 계층을 분리시킴으로써, 코드 재활용성을 높이고 중복 최소화.

![pattern](http://quarl894.github.io/assets/posts/20171206/mvc.png)


## MVC

모든 입력이 Controller에서 처리된다. 입력이 Controller로 들어오면 입력에 해당하는 Model을 조작하고, Model을 나타낼 View를 선택한다.
Controller는 view를 선택할 수 있어 하나의 Controller에서 여러개의 View를 선택할 수 있다.
Controller는 View를 선택만 할 뿐 조작을 하지 않는다. 조작은 Model에서 해준다.

### 장점
- 하나의 Model로 여러개의 View를 만들 수 있다.

### 단점
- 의존성 : View와 Model을 이용하기 때문에 서로간의 의존성을 완벽히 피할 수 없다.
- 테스트 용이성 : Controller가 안드로이드 API에 깊게 종속되므로 유닛 테스트가 어렵다.
- 모듈화 및 유연성 : View를 바꾸기 위해서는 Controller로 돌아가서 Model에게 명령해야하므로 유연성이 떨어진다.
- 유지보수 : 시간이 지남에 따라 Controller부분의 코드가 비대해져 문제가 발생하기 쉽다.

## MVP
MVC패턴에서 Controller의 문제를 해결해주기 위해 등장한 패턴이다.
Controller의 책임에 묶이지 않고 View와 Activity가 자연스럽게 결합하도록 한다.

## View
유일한 변화는 Activity/Fragment가 View에 일부로 간주되는 것이다.

### Prsenter
본질적으로는 MVC의 컨트롤러와 같지만, 뷰에 연결되는 것이 아니라 그냥 인터페이스라는 점이 다르다. 이에 따라 MVC가 가진 테스트 가능성 문제와 함께 모듈화/유연성 문제 역시 해결합니다. 극단적인 MVP를 따르는 살마들은 프리젠터가 절대로 어떤 안드로이드 API나 코드라도 참조해서는 안된다고 주장한다. View와 1대1 관계를 가진다. 

### 장점
- 독립성 : Presenter를 통해서 View와 Model을 완벽히 분리해 줌으로써 Model을 따로 알고 있지 않아도 된다.

### 단점
- 의존성 : View와 1대1 관계로 View에 대한 의존성이 강하다.

## MVVM
ViewModel 말 그대로 View를 나타내기 위한 Model이 추가되었다. View보다는 Model과 유사하게 디자인 되며, ViewModel에 의해 View에 데이터가 바인딩된다. MVP와 같이 View에서 입력이 처리된다. 

### 장점
- 의존성 제로 : Data Binding으로 View와의 의존성을 완벽히 분리한다.

### 단점
- 유지 보수 : 뷰가 변수와 표현식 모두에 데이터 바인딩되어 관계없는 로지기 늘어날 수 있다.

[jekyll-gh]:   https://github.com/quarl894
