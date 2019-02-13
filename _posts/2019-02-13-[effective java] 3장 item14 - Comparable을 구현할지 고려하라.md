---
layout: post
title: "[effective java] 3장 item14 - Comparable을 구현할지 고려하라"
tags: [effective java, study]
comments: true
---

Comparable 인터페이스의 유일무이한 메서드인  compareTo를 알아보자.

### Object equals와 다른점 ?
compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다. 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다.

```java
Arrays.sort(a); // comparable을 구현한 객체들의 배열 정렬
```

순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

compareTo 메서드 작성 요령은 equals와 비슷하다.
Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다.(확인하거나 형변환 필요X)

compareTo 메서드는 필드가 동치인지 비교하는게 아니라 *그 순서를 비교한다.*

compareTo 메서드에서 정수 기본 타입 필드를 비교시 ->
자바 7부터 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare를 이용하면 된다.
관계 연산자 < >를 사용하는 이전 방식은 거추장스럽고 오류를 유발한다.

핵심 필드가 여러개라면 어느것을 먼저 비교하느냐가 중요해진다. **가장 핵심적인 필드부터 비교하자.**

Comparator는 수많은 보조 생성 메서드들로 중무장하고 있다. 자바의 숫자용 기본 타입을 모두 커버한다.
