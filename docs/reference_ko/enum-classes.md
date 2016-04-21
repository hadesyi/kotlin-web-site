---
type: doc
layout: reference
category: "Syntax"
title: "Enum 클래스"
---

# Enum 클래스

enum 클래스의 기본적인 용법은 타입에 안전한 열거형을 구현하는 것이다.

``` kotlin
enum class Direction {
  NORTH, SOUTH, WEST, EAST
}
```

각 열거 상수는 객체이다. 열거 상수는 콤마로 구분한다.

## 초기화

각 열거 상수는 enum 클래스의 인스턴스이므로, 초기화할 수 있다.

``` kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

## 임의 클래스

열거 상수는 임의 클래스를 이용해서 각자 선언할 수 있다.

``` kotlin
enum class ProtocolState {
  WAITING {
    override fun signal() = TALKING
  },

  TALKING {
    override fun signal() = WAITING
  };

  abstract fun signal(): ProtocolState
}
```

베이스 메서드를 오버라이딩하는 것 외에 자신의 메서드를 상응하는 메서드를 가질 수 있다.
enum 클래스가 멤버를 정의하면, 자바처럼 세미콜론을 이용해서 멤버 정의와 enum 상수 정의를 구분해야 한다.

## enum 상수로 작업하기

자바와 마찬가지로, enum 클래스는 정의한 enum 상수 목록을 구하고 이름으로 enum 상수에 접근할 수 있는
메서드를 제공하고 있다. 이 메서드의 시그너처는 다음과 같다(enum 클래스의 이름이 `EnumClass`라고 가정):

``` kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

`valueOf()` 메서드는 지정한 이름에 해당하는 enum 상수가 없으면 `IllegalArgumentException`을 발생한다.

모든 enum 상수는 상수의 이름과 enum 클래스에 정의된 순서를 구할 수 있는 프로퍼티를 갖는다:

``` kotlin
val name: String
val ordinal: Int
```

enum 상수는 또한 [Comparable](/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 인터페이스를 구현하고 있다.
enum 클래스에 정의된 순서를 자연 정렬 순서로 사용한다.
