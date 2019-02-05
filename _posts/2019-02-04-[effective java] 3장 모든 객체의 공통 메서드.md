---
layout: post
title: "[effective java] 3장 모든 객체의 공통 메서드"
tags: [effective java, study]
comments: true
---

object에서 final이 아닌 메서드 (equals, hashCode, toString, clone, finalize)는 모두 재정의overriding을 염두에 두고 설계되었다.

일반 규약에 맞게 재정의 해야 규약을 준수한다고 가정하는 클래스들(HashMap, HashSet등)이 오작동을 하지 않는다.

이번 장에서는 Object메서드들을 언제 어떻게 재정의해야 하는지를 다룬다.

Comparable.compareTo의 Object의 메서드는 아니지만 성격이 비슷하여 함께 다룬다.
