---
layout: post
title: "[effective java] 3장 item10 - equals는 일반 규약을 지켜 재정의하라"
tags: [effective java, study]
comments: true
---

equals 메서드는 재저으이하기 쉬워 보이지만 함정이 도사리고 있다.(젤 쉬운건 재정의 안하기...)

Default는 그 클래스의 인스턴스는 오직 자기 자신과만 같게한다.

So, 다음 상황중 하나에 해당하면 재정의 안하도록..!

- 각 인스턴스가 본질적으로 고유하다.
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
