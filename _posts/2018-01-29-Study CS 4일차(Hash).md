---
layout: post
# title:  "Study CS 4일차(Hash)"
date:   2018-01-29 22:20:00 +0900
categories: Hash, HashTable, HashMap
---

## Hash란 무엇인가?

Hash함수로 만들어진 Hash 값을 Index에 저장되는 자료구조로, Hash값을 통해 접근하는 자료구조. Key-Value 한쌍

## Hash의 장단점을 설명하시오.

Index에 Hash값을 사용함으로써 모든 데이터를 살피지 않아도 검색, 삽입,삭제를 빠르게 수행함.

## 해시 충돌이 무엇이며 왜 일어나는가?

#### 해시충돌

서로 다른 키 값이 해시 함수를 통해 같은 해시 값이 나오는 상황

#### 왜 일어나는 가?

해시함수는 해시값의 개수보다 많은 키값을 해시값으로 변환하기 때문에 서로 다른 키에 대해 동일한 해시값을 내는 경우가 생김.

## Open, Close 해싱 방법이 무엇인가?

1. 충돌이 일어났을 때 특별한 연산을 통해 비어있는 다른 인덱스를 찾는 방법(Open)
2. 배열 안에 배열 혹은 연결 리스트로 계속 붙여나가는 방법(Close)

## HashTable과 HashMap의 공통점과 차이점

#### 공통점

1. Hash 기법을 사용함.
2. Map Inerface를 구현.
3. Key와 Value를 이용해 Data를 관리함.
4. 삽입순서를 유지하지 않음.

#### 차이점

1. 동기화 보장(HashTable은 동기화 보장 / HashMap 동기화 비보장)
2. NULl 허용(Table은 허용함 / Map 비허용)

#### HashMap, HashTable과 HashSet과의 차이점

1. HashSet은 value의 중복을 허용하지 않음.
2. Map Interface 구현에 반해 HashSet은 Set Interface를 구현한다.

## HashTable vs 이진탐색트리

#### 탐색속도 해시테이블 > 이진탐색트리

#### 공간사용 해시테이블 < 이진탐색트리

## Chaning이란 무엇인가?

한 버킷당 들어갈 수 있는 엔트리의 수에 제한을 두지 않음으로써 모든 자료를 해시테이블에 담는 것

## Open Addressing vs Separate Chaining(개방주소법 vs 분리 연결법)

#### 공통점

Worst Case에서 O(M)

#### Open Addressing

특징 : 해시 충돌이 발생하면, (즉 삽입하려는 해시 버킷이 이미 사용 중인 경우) 다른 해시 버킷에 해당 자료를 삽입하는 방식이다.

- Linear Probing
순차적으로 탐색하며 비어있는 버킷을 찾을 때까지 계속 진행된다.

- Quadratic probing
2차 함수를 이용해 탐색할 위치를 찾는다.

- Double hashing probing
하나의 해쉬 함수에서 충돌이 발생하면 2차 해쉬 함수를 이용해 새로운 주소를 할당한다. 위 두 가지 방법에 비해 많은 연산량을 요구하게 된다.

장점 : 충돌이 일어나더라도 주어진 테이블 공간에서 해결한다. 연속된 공간에 데이터를 저장하기 때문에 캐시효율이 더 높다.

단점 : 모든 원소가 반드시 자신의 해시값과 일치하는 주소에 저장된다는 보장을 할 수 없다.

#### Separate Chaning

특징 : 해시 충돌이 잘 발생하지 않도록 보조 해시 함수를 통해 조정할 수 있다면 Worst Case에 가까워 지는 빈도를 줄일 수 있다. Java에서 이 방식으로 HashMap을 구현 중.

- 연결 리스트를 사용하는 방식(Linked List)
각각의 버킷(bucket)들을 연결리스트(Linked List)로 만들어 Collision이 발생하면 해당 bucket의 list에 추가하는 방식이다. 연결 리스트의 특징을 그대로 이어받아 삭제 또는 삽입이 간단하다. 하지만 단점도 그대로 물려받아 작은 데이터들을 저장할 때 연결 리스트 자체의 오버헤드가 부담이 된다. 또 다른 특징으로는, 버킷을 계속해서 사용하는 Open Address 방식에 비해 테이블의 확장을 늦출 수 있다.

- Tree를 사용하는 방식 (Red-Black Tree)
기본적인 알고리즘은 Separate Chaining 방식과 동일하며 연결 리스트 대신 트리를 사용하는 방식이다. 연결 리스트를 사용할 것인가와 트리를 사용할 것인가에 대한 기준은 하나의 해시 버킷에 할당된 key-value 쌍의 개수이다. 데이터의 개수가 적다면 링크드 리스트를 사용하는 것이 맞다. 트리는 기본적으로 메모리 사용량이 많기 때문이다. 데이터 개수가 적을 때 Worst Case를 살펴보면 트리와 링크드 리스트의 성능 상 차이가 거의 없다. 따라서 메모리 측면을 봤을 때 데이터 개수가 적을 때는 링크드 리스트를 사용한다.

장점 : 적재율이 1이 넘어도 사용할 수 있다.

단점 : 해시 테이블의 크기가 M이면 최대 M개 연결리스트가 존재 할 수 있다. 원소 검색 시 연결리스트를 탐색해서 찾아야 한다.

[참조사이트](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/DataStructure#hashtable)

[jekyll-gh]:   https://github.com/quarl894