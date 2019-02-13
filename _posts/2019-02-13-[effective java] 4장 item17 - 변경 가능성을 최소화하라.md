---
layout: post
title: "[effective java] 4장 item17 - 변경 가능성을 최소화하라"
tags: [effective java, study]
comments: true
---

불변 클래스
: 인스턴스의 내부 값을 수정할 수 없는 클래스. 객체가 파괴되는 순간까지 절대 달라지지 않음.
ex) String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal

불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다

### 클래스를 불변으로 만드는 규칙

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 final로 선언한다.
- 모든 필드를 private으로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

```java
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    // 주목할 부분 두가지
    public Complex plus(Complex c) {  // 동사 add가 아닌 전치사 plus 사용
        // 사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complecx 인스턴스를 만들어 반환하는 모습 주목
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

피연산자에 함수를 적용해 결과를 반환하지만 피연산자 자체는 그대로인 프로그래밍 = 함수형 프로그래밍

메서드에서 피연산자인 자신을 수정해 상태가 변하는 것 = 절차적, 명령형 프로그래밍

함수형 프로그래밍을 하면 코드에서 불변이 되는 영역의 비율이 증가하는 장점이 있고, **불변 객체는 단순하다.**

가변 객체는 임의의 복잡한 상태에 놓일 수 있다.

**불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.**

여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.(클래스를 스레드 안전하게 만드는 가장 쉬운방법!)
So, 불변객체는 안심하고 공유할 수 있다:)

적정 팩터리(야이템1)를 제공할 수 있는데, 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.

불변 객체를 자유롭게 공유할 수 있다는점은 방어적복사(아이템50)도 필요없다는 결론으로 이어진다.

**불변 객체끼리는 내부 데이터를 공유할 수 있다.**
**불변 객체들을 구성요소로 사용하면 불변식을 유지하기 수월하다는 이점**
**불변 객체는 그 자체로 실패 원자성을 제공한다.**

### 단점?

값이 다르면 반드시 독립된 객체로 만들어야 한다. 값의 가짓수가 많다면....

### 정리

- getter가 있다고 해서 무조건 setter를 만들지 말자. **클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.**
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
- 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.

확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안 된다.

객체를 재활용할 목적으로 상태를 다시 초기화하는 메서드도 안 된다. -> 복잡성만 커지고 성능 이점은 거의 없다.
