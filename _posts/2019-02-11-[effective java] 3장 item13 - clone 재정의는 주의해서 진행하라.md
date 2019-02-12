---
layout: post
title: "[effective java] 3장 item13 - clone 재정의는 주의해서 진행하라"
tags:
  - effective java
  - study
comments: true
published : false
---

### 메서드 하나 없는 Clonable 인터페이스는 무얼하나?

놀랍게도 Object의 protected 메서드인 clone의 동작 방식을 결정한다.

### clone 메서드의 일반 규약 (허술함)

객체의 복사본을 생성해 반환한다.

### Clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.



### 요약하자면..
- Cloneable을 구현하는 모든 클래스는 clone을 재정의하고 동기화해야한다.
-> 객체의 내부 '깊은 구조'에 숨어 있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 한다.
how? 주로 clone을 재귀적으로 호출해 구현하지만 항상 최선은 아니다.
