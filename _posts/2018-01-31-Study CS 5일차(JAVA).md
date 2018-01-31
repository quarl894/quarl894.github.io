---
layout: post
# title:  "Study CS 5일차(JAVA)"
date:   2018-01-31 22:20:00 +0900
categories: JAVA, CS, 기술면접
---

## Generic의 장점은 무엇인가?

1. 컴파일 시 타입체크를 해줌으로써 타입 안정성을 높임.
2. 형변환의 번거로움을 줄임.
3. API 설계시 명확한 의사전달.

## Overriding vs Overloading

#### Overriding

- 상위 클래스에서 상속받은 메소드를 하위 클래스에서 요구에 맞게 재정의하는 것.

#### Overloading

- 동일한 메소드명을 가진 메소드를 매개변수만 바꿔 사용하는 것.

## 자바의 접근범위

- public
 어떤 클래스에서라도 접근이 가능하다.

- protected
 클래스가 정의되어 있는 해당 패키지 내 그리고 해당 클래스를 상속받은 외부 패키지의 클래스에서 접근이 가능하다.

- default
 클래스가 정의되어 있는 해당 패키지 내에서만 접근이 가능하도록 접근 범위를 제한한다.

- private
 정의된 해당 클래스에서만 접근이 가능하도록 접근 범위를 제한한다.

## ThreadLocal 이란 무엇인가?

스레드 사이에 간섭이 없어야 하는 데이터에 사용한다.

멀티스레드 환경에서는 클래스의 필드에 멤버를 추가할 수 없고 매개변수로 넘겨받아야 하기 때문이다.

==즉, 스레드 내부의 싱글톤을 사용하기 위해 사용한다.==

사용용도 : 사용자 인증, 세션 정보, 트랜잭션 컨텍스트에 사용한다.

주의사항 : 스레드 풀 환경에서 ThreadLocal을 사용하는 경우 ThreadLocal 변수에 보관된 데이터의 사용이 끝나면 반드시 해당 데이터를 삭제해 주어야 한다. 그렇지 않을 경우 재사용되는 쓰레드가 올바르지 않은 데이터를 참조할 수 있다.

## Garbage Collection이란 무엇이고 요청하는 메소드는 무엇인가?

- 가비지를 회수하여 사용할 수 있는 메모리 공간을 늘리는 것.
- 가비지 컬렉션을 수행하는 것을 가비지 컬렉터라고 함.

-  System.gc() or Runtime.getRuntime().gc();

  ++단, 카비지 컬렉션을 요청한다해서 실행이 되는 것은 아님. 실행 판단은 JVM에서 판단함.++

## JVM의 역할에 대해 설명하시오.

#### 정의

JVM이란 JAVA Virtual Machine, 자바 가상 머신의 약자를 따서 줄여 부르는 말이다.(스택기반 가상머신)

#### 역할

1. 자바 어플리케이션을 클래스 로더를 통해 읽어 들여 자바 API와 함께 실행하는 것.
2. JAVA와 OS사이의 중개자 역할을 하여 JAVA가 OS에 구애받지 않고 재사용을 가능하게 해준다.
3. 메모리 관리, Garbage Collection을 수행한다.

## 자바 프로그램의 실행과정

1. 프로그램이 실행되면 JVM은 OS로부터 이 프로그램이 필요로 하는 메모리를 할당받는다.
2. 자바 컴파일러가 자바소스코드를 읽어들여 자바 바이트코드로 변환시킨다.
3. Class Loader를 통해 class 파일들을 JVM으로 로딩한다.
4. 로딩된 class 파일들은 Execution engine을 통해 해석된다.
5. 해석된 바이트코드는 Runtime Data Areas에 배치되어 실질적인 수행이 이루어진다.
6. 이러한 실행과정속에서 JVM은 필요에 따라 Thread Synchronization과 GC같은 관리작업을 수행한다.

## Collection의 종류와 특징을 설명하시오.

#### Collection을 사용하는 이유

- 다수의 Data를 다루는데 표준화된 클래스들을 제공해주기 때문에 편하게 DataStructure를 직접 구현하지 않고 사용할 수 있는 것
- 배열과는 다르게 객체를 보관하기 위한 공간을 미리 정하지 않아도 되므로, 동적할당이 가능하여 공간 효율성 또한 높여준다.

#### Collection 종류

1. List
2. Map
3. Set
4. Stack & Queue

[참조사이트 ](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Java#overriding-vs-overloading)

[jekyll-gh]:   https://github.com/quarl894