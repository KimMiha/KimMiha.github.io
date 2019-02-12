---
layout: post
title: "[effective java] 4장 item16 - public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라"
tags:
  - effective java
  - study
comments: true
---

## public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

```java
// 이처럼 퇴보한 클래스는 public이어서는 안 된다 !
// 데이터 필드에 직접 접근할 수 있어 캡슐화의 이점 제공 못함
class Point{
  public double x;
  public double y;
}
```
API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식도 보장할 수 없고, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다...

public 클래스의 필드가 불변인경우, 불변식만 보장할뿐 나머지 단점들은 갖고있게된다.

```java
// 접근자와 변경자 메서드를 활용해 데이터를 캡슐화한다.
// public 클래스라면 이 방식이 확실히 맞다 !
class Point{
  private double x;
  private double y;
  
  public Point(double x, double y){
    this.x = x;
    this.y = y;
  }
  
  public double getX() { return x; }
  public double getY() { return y; }
  
  public void setX( double x ) { this.x = x; }
  public void setX( double y ) { this.y = y; }
}
```
**패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함**으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

package-private 클래스 혹은 private중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다. 추상 개념만 올바르게 표현해준다면.

why? 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현방식을 바꿀 수 있어서.

**!** 규칙을 어긴 사례 : java.awt.package 의 Point 와 Dimension. 흉내X 타산지석으로 삼길..
