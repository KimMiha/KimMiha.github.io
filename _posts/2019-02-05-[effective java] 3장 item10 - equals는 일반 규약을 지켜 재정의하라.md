---
layout: post
title: "[effective java] 3장 item10 - equals는 일반 규약을 지켜 재정의하라"
tags: [effective java, study]
comments: true
---

equals 메서드는 재정의하기 쉬워 보이지만 함정이 도사리고 있다.(젤 쉬운건 재정의 안하기...)

Default는 그 클래스의 인스턴스는 오직 자기 자신과만 같게한다.

So, 다음 상황중 하나에 해당하면 재정의 안하도록..!

- 각 인스턴스가 본질적으로 고유하다.
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

equals가 실수로라도 호출되는걸 막고싶다면
```java
@Override public boolean equals(Object o){
    throw new AssertionError(); //호출금지
  }
```

### 그렇다면 재정의 해야 할때?

- 객체 식별성이 아니라 논리적 동치성을 확인해야 할 때

ex) 값 클래스들(Integer, String...) 값이 같은지를 알고싶어 할테니까 !

but, 값 클래스여도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제클래스라면 재정의 안해도 OK.

ex)Enum   논리적 동치성 = 객체 식별성


### Object 명세에 적힌 규약

  equals 메서드는 동치관계를 구현하며 다음을 만족한다.
  - 반사성 : null이 아닌 모든 참조 값 x에 대해 x.equals(x) 는 true다.
  - 대칭성 : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
  - 추이성 : null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true 이고 y.equals(z) 도 true면 x.equals(z)도 true다.
  - 일관성 : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 같은 결과를 반환한다.
  - null 아님 : null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.
  
  동치관계 : 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산이다.
  
  equals메소드가 쓸모있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.
  
